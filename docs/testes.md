# Casos de teste e resultados

Testes feitos no ambiente de teste em 25 e 26/11/2026. As funções de teste ficam em [`test/TSTSP11.tlpp`](../test/TSTSP11.tlpp) e **não vão para produção**.

**Como executar as funções de teste:** compile o `TSTSP11.tlpp` (ele usa `using namespace custom.superparametros.model`), entre em um **módulo** (logado na empresa e filial onde a WFU existe) e chame a função pelo menu. Não execute como programa inicial: sem ambiente aberto, aparece o erro "Alias does not exist: WFU".

## 1. Resumo

| # | Caso | Regras | Resultado |
|---|---|---|---|
| 1 | Inclusão como usuário comum | RN27, RN28 | ✅ 25/11 |
| 2 | Alteração: campos de expressão e modo de edição | RN28 | ✅ 25/11 |
| 3 | Parâmetro em Modo Admin | RN25 | ✅ 25/11 |
| 4 | Linha sem chave e sem grupo, ou com os dois | RN31 | ✅ 25/11 |
| 5 | Validação do item (`WFU_VLDITE`) | RN20 | ✅ 25/11 |
| 6 | Validação do item com erro na expressão | RN20, RN39 | ✅ 26/11 (reteste) |
| 7 | Validação de ativação ao salvar | RN22, RN39 | ✅ 26/11 (reteste) |
| 8 | Regras da linha de grupo | RN32 a RN34, RN36, RN37 | ✅ 25/11 |
| 9 | Apelido do grupo na linha | RN38 | ✅ 25/11 |
| 10 | Exclusão de parâmetro usado como grupo | RN35 | ✅ 25/11 |
| 11 | Consulta (`GetParamValue` e `ValidEntityParamList`) | RN16 a RN19, RN21, RN29, RN30 | ✅ 25/11 |
| 12 | Erro no `WFU_VLDATV` durante a consulta, com e sem tela | RN23, RN24 | ✅ 26/11 |
| 13 | `GetParamValue` sem o `x_Default` | RN40 | ✅ 25/11 |
| 14 | Tipo Lógico com V/F | RN06, RN42 | ✅ 26/11 |
| 15 | Usuário comum excluindo em Edição Livre | RN26 | ✅ 26/11 |
| 16 | Validação do conteúdo ao sair do campo e limpeza ao trocar o tipo | RN41 | ✅ 26/11 |

## 2. Casos

### 1. Inclusão como usuário comum
- **Passos:** incluir um parâmetro logado como usuário comum.
- **Esperado:** `WFU_MODEDI` nasce como `1 - Edição Livre` e bloqueado; `WFU_VLDITE` e `WFU_VLDATV` liberados.
- **Resultado:** conforme o esperado.

### 2. Alteração
- **Passos:** alterar um parâmetro como usuário comum e depois como administrador.
- **Esperado:** para o usuário comum, `WFU_VLDITE`, `WFU_VLDATV` e `WFU_MODEDI` bloqueados; para o administrador, liberados.
- **Resultado:** conforme o esperado.

### 3. Modo Admin
- **Passos:** com o parâmetro em `0 - Modo Admin`, tentar alterar e excluir como usuário comum e como administrador.
- **Esperado:** usuário comum bloqueado com mensagem; administrador acessa.
- **Resultado:** conforme o esperado.

### 4. Chave ou grupo
- **Passos:** linha com chave e grupo vazios; linha com os dois preenchidos.
- **Esperado:** as duas bloqueadas.
- **Resultado:** conforme o esperado.

### 5. Validação do item
- **Preparação:** parâmetro `TSTVLDITE`, tipo Caractere, `WFU_VLDITE` = `Len(AllTrim(FwFldGet("WFV_CHBUSC"))) == 6`.
- **Esperado:** chave com tamanho diferente de 6 recusada (erro do usuário); `000001` aceita; `*` aceita (não passa pela validação).
- **Resultado:** `00000001` (8 caracteres) recusada corretamente.
- **Observação:** com `ExistCpo("SA1", FwFldGet("WFV_CHBUSC"))` o cliente foi recusado, porque o `FwFldGet` devolve a chave com espaços. Use `ExistCpo("SA1", AllTrim(FwFldGet("WFV_CHBUSC")))`.

### 6. Validação do item com erro na expressão
- **Passos:** como administrador, informar no `WFU_VLDITE`:
  - `Len(AllTrim(FwFldGet("WFV_CHBUSC")) == 6` (parêntese faltando);
  - `"ABC"` (com aspas: texto, não lógico);
  - `ABC` (sem aspas: variável inexistente).
- **Esperado:** linha bloqueada, mensagem de erro interno com o erro real, tela aberta.
- **Resultado 25/11:** bloqueou, mas a mensagem escondia o erro ("devolveu o tipo U"). Causa: o `ErrorBlock` + `Break` não capturava o erro no TLPP. Corrigido na `v.25112026` com `try/catch` (RN39).
- **Resultado 26/11 (reteste):** todos bloqueados com a mensagem certa.

### 7. Validação de ativação ao salvar
- **Passos:** parâmetro `TSTVLDATV`, tipo Lógico, conteúdo `V`, com `WFU_VLDATV`:
  - `Day(dDataBase) = = 31` → bloqueia;
  - `"ABC"` → bloqueia (não lógico);
  - `Day(dDataBase) != 99` → grava;
  - `Day(dDataBase) == 99` → grava (falso é válido ao salvar).
- **Resultado 25/11:** comportamento certo, mesmo defeito de mensagem do caso 6.
- **Resultado 26/11 (reteste):** bloqueou com a mensagem certa.

### 8. Linha de grupo
- **Esperado:** bloqueia linha de grupo com conteúdo, com o próprio parâmetro, com código inexistente, com grupo inativo, com grupo que tem grupo e o mesmo grupo duas vezes. Bloqueia inativar parâmetro usado como grupo.
- **Resultado:** todos conforme o esperado.

### 9. Apelido do grupo
- **Esperado:** ao informar o grupo, aparece o apelido; ao reabrir o parâmetro, o apelido aparece nas linhas gravadas.
- **Resultado:** conforme o esperado nos dois casos.

### 10. Exclusão de parâmetro usado como grupo
- **Esperado:** bloqueada, informando qual parâmetro o usa.
- **Resultado:** conforme o esperado.

### 11. Consulta
- **Preparação:**

  | Apelido | Tipo | Conteúdo padrão | Itens |
  |---|---|---|---|
  | `USUDEPTO` | Caractere | `X` | chave `000001` |
  | `BLQARMAZEM` | Lógico | `V` | chave `000099` = `F`; linha de grupo com o código do `USUDEPTO` |

- **Passos:**
  1. `U_TSTSP11` com o parâmetro valendo.
  2. Inativar o `BLQARMAZEM` e executar a `U_TSTSP11B`.
  3. Reativar, informar `WFU_VLDATV` = `Day(dDataBase) == 99` e executar a `U_TSTSP11B`.
- **Esperado:**

  | Chamada | Valendo | Inativo ou `VLDATV` falso |
  |---|---|---|
  | `GetParamValue("BLQARMAZEM", "000099", .F.)` | `.F.` (exceção) | `.F.` |
  | `GetParamValue("BLQARMAZEM", "000001", .F.)` | `.T.` (pelo grupo) | `.F.` |
  | `GetParamValue("BLQARMAZEM", "000777", "PADRAO")` | `"PADRAO"` (fora da lista: `x_Default`) | |
  | `GetParamValue("BLQARMAZEM", "", .F.)` | `.T.` (sem chave) | `.F.` |
  | `GetParamValue("NAOEXISTE", "000099", .F.)` | `.F.` | |
  | `ValidEntityParamList("BLQARMAZEM", "000099")` | `.T.` | |
  | `ValidEntityParamList("BLQARMAZEM", "000001")` | `.T.` | `.F.` |
  | `ValidEntityParamList("BLQARMAZEM", "000777")` | `.F.` | |
  | `ValidEntityParamList("BLQARMAZEM", "")` | `.F.` | |

- **Resultado:** todas as linhas `OK` nas 3 execuções (repetido em 26/11 com os dados em V/F).
- **Retestar:** em 26/11 a RN17 mudou (fora da lista devolve o `x_Default`). A linha do `000777` foi ajustada na `U_TSTSP11` para usar o `x_Default` `"PADRAO"`. Falta executar de novo.

### 12. Erro no `WFU_VLDATV` durante a consulta
- **Preparação:** parâmetro `TSTERRATV`, tipo Lógico, conteúdo `V`, com `WFU_VLDATV` = `U_TSTSPATV()` (função que só gera erro quando o teste pede; ao salvar, devolve `.T.`).
- **Passos:** executar a `U_TSTSP12`. Ela consulta com tela e depois em job (`U_TSTSP12J`).
- **Esperado:** com tela, mensagem de erro interno e retorno `.F.`; em job, nenhuma tela, a mesma mensagem e "[TSTSP12J] Sem tela - retorno: .F." no `console.log`.
- **Resultado 26/11:** mensagem com o erro real ("Erro forçado pelo teste 12 do Super Parâmetros", função `SuperParametros.CheckActivation`), retorno `.F.` com tela e mensagem no `console.log` do job.
- **Observação:** a primeira execução devolveu `.T.` porque a expressão tinha sido cadastrada no `WFU_VLDITE` em vez do `WFU_VLDATV`.

### 13. `GetParamValue` sem o `x_Default`
- **Passos:** `GetParamValue("BLQARMAZEM", "000099")` (incluído na `U_TSTSP11`).
- **Esperado:** mensagem "O valor padrão de retorno não foi informado..." com a rotina que chamou, e retorno `Nil`.
- **Resultado:** retorno `Nil`.

### 14. Tipo Lógico com V/F
| Passos | Esperado | Resultado |
|---|---|---|
| Cabeçalho tipo Lógico: `V`, depois `f` | Aceita os dois | ✅ |
| Cabeçalho tipo Lógico: `1`, `0` e `X` | Recusa, com "Informe V (verdadeiro) ou F (falso)" | ✅ |
| Item do grid de parâmetro Lógico: `F`, depois `1` | Aceita `F` e recusa `1` | ✅ |
| `U_TSTSP11` com os dados em V/F | Todas as linhas `OK` | ✅ |
| Combo do `WFU_TIPO` | `L - Lógico (V - Verdadeiro / F - Falso)` em uma opção só | ✅ |

### 15. Exclusão em Edição Livre
- **Passos:** usuário comum exclui um parâmetro em `1 - Edição Livre` (sem itens de outra filial e sem uso como grupo).
- **Resultado:** o usuário consegue excluir.

### 16. Validação do conteúdo ao sair do campo (RN41)
| Passos | Esperado |
|---|---|
| Parâmetro tipo Numérico: digitar `abc` no conteúdo padrão e sair do campo | Mensagem na hora ("não é um número válido"), sem esperar o Confirmar |
| Parâmetro tipo Lógico: digitar `X` no conteúdo padrão e sair do campo | Mensagem na hora ("Informe V (verdadeiro) ou F (falso)") |
| Com o conteúdo padrão preenchido, trocar o tipo (ex.: de Numérico para Lógico) | O conteúdo padrão é apagado |
| No grid de um parâmetro Lógico: digitar `1` no conteúdo do item e sair do campo | Mensagem na hora |
| Sair do conteúdo sem mudar o valor | Nenhuma validação nova |
| Linha de grupo com conteúdo vazio | Passa pela validação do campo (a regra de grupo continua ao sair da linha) |

**Resultado 26/11:** todos conforme o esperado.
