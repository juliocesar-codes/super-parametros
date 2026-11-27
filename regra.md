# Padrão de desenvolvimento de código (PRW e TLPP)

Vale para todo fonte `.prw` e `.tlpp`.

## 1. Variáveis

Formato: `<tipo>_<Nome>`

- `<tipo>`: letra minúscula que indica o tipo da variável.
- `<Nome>`: nome da variável, em PascalCase.

Exemplo: `c_Armazem`, `n_Quant`, `l_Ret`.

| Letra | Tipo |
|---|---|
| `c` | Caractere |
| `n` | Numérico |
| `l` | Lógico |
| `d` | Data |
| `a` | Array |
| `o` | Objeto |
| `b` | Bloco de código |
| `x` | Variante (tipo indefinido) |

### Tamanho do nome

| Formato | Regra |
|---|---|
| `.tlpp` | Nome **sempre semântico**, sem limite de tamanho. |
| `.prw` | Limite de **10 caracteres** (o compilador só diferencia os 10 primeiros). |

## 2. Classes, métodos e funções

| Elemento | `.tlpp` | `.prw` |
|---|---|---|
| Classes | Nome semântico. | Não existem. |
| Métodos | Nome semântico. | Não existem. |
| Funções de usuário (`User Function`) | Nome semântico. | Limite de 10 caracteres. |
| Funções estáticas (`Static Function`) | Nome semântico. | Nome semântico. |

## 3. Documentação

Toda função, classe e método leva um bloco `Protheus.doc` no formato abaixo:

```advpl
/*/{Protheus.doc} Nome da função, classe ou método
    @author [Nome de quem criou a função]
    @since [data atual]
    @version 1.0
    @param [Nome da variável], [Tipo da variável], [O que ela faz/é responsável por]
    @return [Nome da variável], [descrição do que retorna]
/*/
```

- `@author`: nome de quem **criou** a função. Em função já existente, **mantém o criador original** (ex.: Júlio César) e nunca o sobrescreve ao editar. Função nova leva o nome do desenvolvedor que a criou.
- `@since`: data em que o fonte foi criado, no formato `dd/mm/aaaa`.
- Um `@param` por parâmetro.
- Função sem parâmetros ou sem retorno omite a linha correspondente.
- As descrições (texto do bloco, `@param` e `@return`) são escritas em português correto, com acentuação e cedilha (ex: "função", "descrição", "parâmetro"). Não escrever sem acento.

## 4. Codificação do arquivo

Todo fonte `.prw` e `.tlpp` é salvo em **Windows-1252** (não UTF-8). Gravado em UTF-8, os acentos aparecem quebrados no editor.

- Ao criar ou editar um fonte, garantir que o arquivo termine em Windows-1252, sem BOM. Ferramentas de escrita costumam gravar em UTF-8, então reconverter após cada edição.
- Conferir relendo o arquivo como Windows-1252: os acentos devem aparecer corretos e não pode haver `?` no lugar deles.

## 5. Mensagens

Vale para toda mensagem do código: erro, alerta, sucesso, informação ou qualquer outra. Isso inclui as exibidas ao usuário (`Help`, `FWAlertError`, `FWAlertWarning`, `FWAlertSuccess`, `FWAlertInfo`, `MsgAlert`, `MsgStop`, `MsgInfo`...) e as registradas em log (`ConOut`).

### 5.1 Versão do código

- A versão fica em uma constante no topo do fonte, com o nome `[NOME_FONTE]_VERSAO` (ex.: fonte `PARAMCAD` → `PARAMCAD_VERSAO`).
- O valor é sempre `"v.DDMMAAAA"`, com a **data atual sem as barras** (ex.: 23/11/2026 → `"v.23112026"`).
- Ao alterar o fonte, atualizar a constante para a data do dia.
- Se a constante não existir, criá-la e aplicá-la nas mensagens do fonte.

```advpl
#define PARAMCAD_VERSAO "v.23112026"                    // Versão do código (v.DDMMAAAA)
#define PARAMCAD_TITULO "PARAMCAD - " + PARAMCAD_VERSAO  // Título padrão das mensagens
```

### 5.2 Título

O título de **toda** mensagem contém a versão do código, no formato `NOME_FONTE - v.DDMMAAAA`. Recomenda-se definir uma constante `[NOME_FONTE]_TITULO` (como no exemplo acima) e usá-la em todas as mensagens.

### 5.3 Corpo

- Diz **o que aconteceu** e, sempre que possível, **por quê**, com dados concretos (valor digitado, item, filial, campo).
- Informa o **tipo**: `Erro do usuário` (dado inválido, regra de negócio, falta de permissão) ou `Erro interno` (falha do sistema, dicionário ou configuração ausente, exceção inesperada).
- **Erro do usuário:** orienta como corrigir.
- **Erro interno:** orienta a contatar o departamento de T.I. e informa o nome da função.
- Escrita em português correto, com acentuação e cedilha.

Exemplo:

```advpl
Help(Nil, Nil, PARAMCAD_TITULO, Nil, "O conteúdo [abc] não é um número válido para o tipo Numérico." + CRLF + "Tipo: erro do usuário.", 1, 0, Nil, Nil, Nil, Nil, Nil, {"Informe um número, usando ponto como separador decimal (ex.: 12.5)."})
```

### 5.4 Código existente

- Fonte com mensagens e **sem** a constante: criar a constante e aplicá-la nas mensagens.
- Fonte com a constante mas com mensagens **fora do padrão**: ajustar as mensagens ao padrão desta seção.
- Quando for possível e realmente necessário, melhorar a mensagem (mais clara, com o motivo e dados concretos). Não alterar a regra de negócio por causa da mensagem.
- Fonte sem mensagens não precisa da constante.
