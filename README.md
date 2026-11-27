# Super Parâmetros

Parâmetros customizados para Protheus em tabelas próprias, com tela de cadastro e uma classe de consulta para os programas.

Cada parâmetro tem um **valor padrão** e pode ter uma **lista de exceções** por chave de busca (ex.: código do usuário). Um parâmetro pode usar a lista de outro como **grupo**, o que evita cadastros repetidos: 7 usuários com permissão em 10 armazéns, que na SX6 seriam 70 parâmetros, viram 2 cadastros.

**Versão atual dos fontes:** `v.26112026`

## Funcionalidades

- **Valor padrão e exceções:** conteúdo padrão no parâmetro e conteúdo próprio por chave. A chave `*` vale para todas.
- **Tipos:** Caractere, Numérico (ponto decimal), Lógico (`V`/`F`) e Data (`DD/MM/AAAA`), validados no cadastro e convertidos na consulta.
- **Grupos:** uma linha da lista pode apontar para outro parâmetro, incluindo todos os itens dele.
- **Validação do item:** expressão (macro) que confere a chave digitada (ex.: o cliente precisa existir).
- **Validação de ativação:** expressão (macro) que decide se o parâmetro vale naquele momento (ex.: não vale no último dia do mês).
- **Modo de edição:** Edição Livre (todos editam) ou Modo Admin (só administradores).
- **Controle por filial:** a tabela de super parâmetro é compartilhada; a tabela de itens do super parâmetro segue a regra da empresa.
- **Robustez:** expressões rodam protegidas por `try/catch`. Um erro vira mensagem (ou log, em job) e nunca derruba a rotina.

## Estrutura do repositório

Uma pasta por camada, uma classe por arquivo e o namespace acompanhando a pasta (detalhes em [arquitetura_e_banco.md](docs/arquitetura_e_banco.md)).

```text
super_parametros/
├── README.md
├── src/
│   ├── view/
│   │   └── PARAMCAD.tlpp             Tela MVC: browse, MenuDef, ModelDef, ViewDef
│   ├── service/
│   │   └── ParamCadService.tlpp      Regras do cadastro
│   ├── model/
│   │   └── SuperParametros.tlpp      Consulta usada pelos programas
│   ├── repository/
│   │   └── ParamRepository.tlpp      Acesso a dados (WFU/WFV) do cadastro
│   └── util/
│       ├── ExpressionRunner.tlpp     Execução protegida das expressões (macro)
│       ├── ParamMessage.tlpp         Exibição das mensagens
│       └── SPConstantes.tlpp         Constantes compartilhadas
├── test/
│   └── TSTSP11.tlpp                  Funções de teste (só no ambiente de teste)
└── docs/
    ├── requisitos_negocio.md         Regras de negócio (RN)
    ├── requisitos_funcionais.md      Requisitos funcionais (RF) e fluxo da consulta
    ├── requisitos_nao_funcionais.md  Requisitos não funcionais (RNF)
    ├── arquitetura_e_banco.md        Organização dos fontes, diagrama de classes e banco
    └── testes.md                     Casos de teste e resultados
```

| Camada | Namespace |
|---|---|
| view | `custom.superparametros.view` |
| service | `custom.superparametros.service` |
| model | `custom.superparametros.model` |
| repository | `custom.superparametros.repository` |
| util | `custom.superparametros.util` |

## Instalação

1. **Dicionário:** crie ou ajuste as tabelas, os índices e a consulta padrão, conforme [arquitetura_e_banco.md](docs/arquitetura_e_banco.md). No combo do `WFU_TIPO`, não use `;` dentro da descrição de uma opção, porque ele separa as opções.
2. **Constantes:** `SPConstantes` está em `src/util/`; não é necessário instalar um arquivo `.ch`.
3. **Compilação:** compile `src/util/SPConstantes.tlpp` primeiro e, depois, os demais fontes de `src/` no mesmo RPO.
4. **Menu:** a rotina fica em `custom.superparametros.view.U_PARAMCAD`. Inclua (ou atualize) a `U_PARAMCAD` no menu pelo Configurador com esse namespace. Ex: `custom.superparametros.view.U_PARAMCAD`
5. **Administradores:** o Modo Admin usa o `FWIsAdmin()`. Confira quem pertence ao grupo de administradores.

Não leve o `test/TSTSP11.tlpp` para produção.

## Uso nos programas

```advpl
#include "totvs.ch"

using namespace custom.superparametros.model

User Function MinhaRotina()

    Local o_SP     As Object
    Local n_Desc   As Numeric
    Local l_Bloq   As Logical

    o_SP := SuperParametros():New()

    // Valor do parâmetro para a chave, já convertido pelo tipo cadastrado.
    // O 3º argumento (x_Default) é obrigatório: é o retorno quando o parâmetro
    // não existe, está inativo ou a validação de ativação deu falso/erro.
    n_Desc := o_SP:GetParamValue("DESCMAX", SA1->(A1_COD + A1_LOJA), 0)

    // A chave faz parte da lista do parâmetro (direto, pelo "*" ou por um grupo)?
    l_Bloq := o_SP:ValidEntityParamList("BLQARMAZEM", __cUserId)

Return
```

| Método | Retorno |
|---|---|
| `GetParamValue(apelido, chave, x_Default)` | Valor convertido pelo tipo. Chave na lista devolve o conteúdo do item (ou o conteúdo padrão, se o item estiver vazio ou a chave vier de um grupo). Chamada sem chave ou parâmetro sem itens devolvem o conteúdo padrão. Chave fora da lista, parâmetro inexistente, inativo ou com validação de ativação falsa devolvem o `x_Default`. |
| `ValidEntityParamList(apelido, chave)` | `.T.` se a chave está na lista (exata, `*` ou grupo) ou se o parâmetro não tem itens. |

O fluxo completo está em [requisitos_funcionais.md](docs/requisitos_funcionais.md#fluxo-da-consulta-getparamvalueapelido-chave-x_default).

### Exemplo: permissão de armazéns com grupo

```text
USUARM  (grupo, tipo C)           PERMARM (tipo C, conteúdo padrão = "01,02,03,04,05,06,07,08,09,10")
  CHBUSC   CONTEU                   CHBUSC   IDPAR    CONTEU
  000001   (vazio)                  (vazio)  USUARM   (vazio)
  ...      (os 7 usuários)
```

Um usuário do grupo recebe o conteúdo padrão do `PERMARM` (os 10 armazéns). Quem está fora da lista recebe o `x_Default` (aqui, `""`), então não tem permissão em nenhum armazém:

```advpl
a_Armazens := StrTokArr(AllTrim(o_SP:GetParamValue("PERMARM", __cUserId, "")), ",")
l_Permite  := AScan(a_Armazens, {|c_Arm| AllTrim(c_Arm) == AllTrim(c_Local)}) > 0
```

Para o parâmetro valer para todos, cadastre a chave `*` na lista.

Não use `c_Local $ c_Armazens`: o `$` procura texto dentro de texto, então "1" seria encontrado em "10".

## Expressões (macro)

| Campo | Exemplo | Efeito |
|---|---|---|
| Validação do item (`WFU_VLDITE`) | `ExistCpo("SA1", AllTrim(FwFldGet("WFV_CHBUSC")))` | A chave precisa ser um cliente existente. |
| Validação do item (`WFU_VLDITE`) | `Len(AllTrim(FwFldGet("WFV_CHBUSC"))) == 6` | A chave precisa ter 6 caracteres. |
| Validação de ativação (`WFU_VLDATV`) | `Day(dDataBase) != 31` | O parâmetro não vale no dia 31. |
| Validação de ativação (`WFU_VLDATV`) | `dDataBase != LastDay(dDataBase)` | Não vale no último dia do mês. |

- Use sempre `AllTrim(FwFldGet("WFV_CHBUSC"))`: o `FwFldGet` devolve a chave com espaços até o tamanho do campo.
- A expressão precisa devolver `.T.` ou `.F.`.
- A validação de ativação roda ao salvar (como teste) e em cada consulta. Não use expressões que gravem dados nem que abram e posicionem tabelas.

## Documentação

- [Requisitos de negócio](docs/requisitos_negocio.md)
- [Requisitos funcionais](docs/requisitos_funcionais.md)
- [Requisitos não funcionais](docs/requisitos_nao_funcionais.md)
- [Arquitetura e estrutura do banco](docs/arquitetura_e_banco.md)
- [Casos de teste e resultados](docs/testes.md)
- Padrão de código: `./regra.md` (fora desta pasta)

## Pendências

| Item | Situação |
|---|---|
| **Consulta pelo repositório** | A `SuperParametros` ainda faz as próprias buscas na WFU/WFV. Movê-las para o `ParamRepository` é melhoria futura, com novo teste da consulta. |

## Autores

- Gabriel Lacerda Correia
- Julio César
"# super-parametros" 
