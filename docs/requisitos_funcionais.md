# Requisitos funcionais (RF)

O que o sistema faz. As regras que cada requisito segue estão em [requisitos_negocio.md](requisitos_negocio.md).

**Situação:** ✅ implementado e testado

## Cadastro (tela PARAMCAD)

| ID | Requisito | Situação |
|---|---|---|
| RF01 | Cadastrar parâmetros: visualizar, incluir, alterar e excluir. | ✅ |
| RF02 | Listar os parâmetros em um browse com pesquisa e legenda (verde = ativo, vermelho = inativo). | ✅ |
| RF03 | Informar no parâmetro: código (automático), apelido, descrição, tipo, conteúdo padrão, ativo, validação do item, validação de ativação e modo de edição. | ✅ |
| RF04 | Cadastrar na mesma tela a lista de itens (grid): item numerado automaticamente, chave de busca, parâmetro grupo (com F3), apelido do grupo e conteúdo. | ✅ |
| RF05 | Permitir salvar o parâmetro sem nenhum item. | ✅ |

## Validações

| ID | Requisito | Situação |
|---|---|---|
| RF06 | Validar o apelido: obrigatório, sem espaços e sem repetição. | ✅ |
| RF07 | Validar o conteúdo (do parâmetro e dos itens de chave) conforme o tipo, inclusive o Lógico com `V`/`F`. | ✅ |
| RF08 | Exigir chave ou grupo em cada linha (nunca os dois), sem repetir chave nem grupo. | ✅ |
| RF09 | Permitir item de chave com conteúdo vazio, que usa o padrão do parâmetro. | ✅ |
| RF10 | Executar a validação do item (`WFU_VLDITE`) ao sair da linha. | ✅ |
| RF11 | Testar a validação de ativação (`WFU_VLDATV`) ao salvar. | ✅ |
| RF12 | Validar a linha de grupo: sem conteúdo, grupo existente, ativo, diferente do próprio parâmetro e sem grupo dentro de grupo. | ✅ |
| RF23 | Validar o conteúdo pelo tipo ao sair do campo (cabeçalho e grid) e apagar o conteúdo padrão quando o tipo for trocado. | ✅ |

## Controle de acesso

| ID | Requisito | Situação |
|---|---|---|
| RF13 | Bloquear alterar ou excluir parâmetro em Modo Admin para quem não é administrador, antes de abrir a tela. | ✅ |
| RF14 | Bloquear para usuário comum os campos `WFU_VLDITE` e `WFU_VLDATV` na alteração, e o `WFU_MODEDI` sempre. | ✅ |
| RF15 | Mostrar na lista apenas os itens da filial logada. | ✅ |
| RF16 | Bloquear alterar ou excluir item de outra filial, com mensagem informando o motivo, a função e a versão. | ✅ |
| RF17 | Bloquear a exclusão do parâmetro quando houver item de outra filial. | ✅ |
| RF18 | Bloquear excluir ou inativar um parâmetro usado como grupo, informando qual parâmetro o usa. | ✅ |

## Consulta (classe `SuperParametros`, usada por outros programas)

| ID | Requisito | Situação |
|---|---|---|
| RF19 | Devolver o valor do parâmetro já convertido para o tipo cadastrado (`GetParamValue`), seguindo o fluxo abaixo. | ✅ |
| RF20 | Informar se uma chave faz parte da lista do parâmetro (`ValidEntityParamList`), incluindo os grupos. | ✅ |
| RF21 | Avaliar a validação de ativação na consulta, com mensagem (com tela) ou log (sem tela) em caso de erro. | ✅ |

## Gravação

| ID | Requisito | Situação |
|---|---|---|
| RF22 | Reservar o número do parâmetro na inclusão, confirmá-lo ao gravar e devolvê-lo se a inclusão for cancelada. | ✅ |

## Fluxo da consulta: `GetParamValue(apelido, chave, x_Default)`

| Passo | Situação | Retorno |
|---|---|---|
| 0 | `x_Default` não informado (mensagem de erro) | `Nil` |
| 1 | Parâmetro não existe | `x_Default` |
| 2 | `WFU_ATIVO = "0"` | `x_Default` |
| 3 | `WFU_VLDATV` preenchido e deu falso ou erro (com mensagem) | `x_Default` |
| 4 | Chamada sem chave | `WFU_CONTEU` |
| 5 | Parâmetro sem itens na filial | `WFU_CONTEU` |
| 6 | Chave exata na lista | `WFV_CONTEU` (se vazio, `WFU_CONTEU`) |
| 7 | `*` na lista | `WFV_CONTEU` (se vazio, `WFU_CONTEU`) |
| 8 | Chave dentro de um grupo | `WFU_CONTEU` |
| 9 | Chave não encontrada (fora da lista) | `x_Default` |

Os retornos dos passos 4 a 8 são convertidos pelo `WFU_TIPO` (Lógico: `V` → `.T.`, qualquer outro valor → `.F.`).

O `ValidEntityParamList(apelido, chave)` devolve:
- `.F.` nos passos 1 a 3;
- `.T.` quando o parâmetro não tem itens;
- `.T.` quando a chave é encontrada nos passos 6 a 8;
- `.F.` nos demais casos, inclusive na chamada sem chave.
