# Requisitos de negócio (RN)

Regras de negócio do Super Parâmetros. Origem: levantamento inicial (18 a 20/11/2026), reunião de 23/11/2026, conversa com o cliente de 24/11/2026 e testes de 25 e 26/11/2026.

**Situação:** ✅ implementado e testado · ⏳ implementado, falta testar · ❌ removida
**Origem:** Existente (levantamento inicial) · Nova (definida de 23 a 26/11)

## Cadastro e estrutura

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN01 | Os parâmetros ficam centralizados nas tabelas WFU (parâmetro) e WFV (itens). | Existente | ✅ |
| RN02 | A WFU é sempre compartilhada: um parâmetro vale para todas as filiais. | Existente | ✅ |
| RN03 | A WFV segue a regra da empresa (compartilhada ou exclusiva por filial). | Existente | ✅ |
| RN04 | O código do parâmetro é sequencial e automático. | Existente | ✅ |
| RN05 | O apelido é obrigatório, sem espaços, único e definido só na inclusão. | Existente | ✅ |
| RN06 | Tipos aceitos: Caractere, Numérico (ponto decimal, ex.: `12.5`), Lógico (`V` = verdadeiro, `F` = falso) e Data (`DD/MM/AAAA`). O conteúdo é validado pelo tipo. | Existente (alterada) | ✅ |
| RN07 | Um único tipo de entidade por parâmetro (Db Alias). Substituída pela RN20. | Existente | ❌ |
| RN08 | A chave do item se chama Chave de Busca (`WFV_CHBUSC`) e não se repete no parâmetro. É obrigatória, exceto na linha de grupo. | Existente (alterada) | ✅ |
| RN09 | Item com conteúdo vazio usa o conteúdo padrão do parâmetro. | Existente | ✅ |
| RN10 | A chave `*` vale para todas as chaves. | Existente | ✅ |
| RN11 | Toda mensagem traz a versão do fonte no título, o motivo e o tipo do erro (usuário ou interno). | Existente | ✅ |
| RN41 | O conteúdo (do parâmetro e do item) é validado pelo tipo quando o usuário sai do campo com um valor diferente do anterior. Trocar o tipo do parâmetro apaga o conteúdo padrão já digitado. | Nova | ✅ |
| RN42 | O conteúdo do tipo Lógico é digitado como `V` ou `F` (maiúscula ou minúscula), no parâmetro e nos itens. O combo do tipo mostra `L - Lógico (V - Verdadeiro / F - Falso)`. | Nova | ✅ |

## Controle por filial

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN12 | O usuário vê só os itens da filial logada, inclusive o administrador. | Existente | ✅ |
| RN13 | Alterar ou excluir um item só é permitido na filial do item, inclusive para o administrador. | Existente | ✅ |
| RN14 | A exclusão do parâmetro é bloqueada se houver item dele em outra filial. | Existente | ✅ |
| RN15 | Filial sem itens (com a WFV exclusiva): o parâmetro vale com o conteúdo padrão para todos nessa filial. | Existente | ✅ |

## Consulta

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN16 | Parâmetro inexistente ou inativo devolve o `x_Default` (valor padrão informado por quem chama). | Existente | ✅ |
| RN17 | Chave fora da lista (sem a chave exata, sem o `*` e fora dos grupos) devolve o `x_Default`: quem não está na lista não recebe o conteúdo. Parâmetro sem itens (RN15) e chamada sem chave (RN18) continuam devolvendo o `WFU_CONTEU`. | Existente (alterada em 26/11) | ⏳ falta retestar |
| RN18 | Chamada sem chave devolve o `WFU_CONTEU`. | Nova | ✅ |
| RN19 | Ordem da busca: chave exata, depois `*`, depois grupos (`WFV_IDPAR`). | Nova | ✅ |
| RN40 | O `x_Default` do `GetParamValue` é obrigatório. Sem ele, aparece uma mensagem de erro interno (ou log, sem tela) com a rotina que chamou, e a função devolve `Nil` sem consultar o parâmetro. | Nova | ✅ |

> **Histórico da RN17:** no levantamento, a chave fora da lista devolvia o `x_Default`. Em 24/11 passou a devolver o `WFU_CONTEU`, o que obrigava a chamar o `ValidEntityParamList` antes em parâmetros de permissão. Em 26/11 voltou a devolver o `x_Default`. Para a lista valer para todos, cadastre a chave `*`.

## Validação do item

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN20 | A chave de busca é validada pela expressão do `WFU_VLDITE` (macro) ao sair da linha. Campo vazio significa sem validação. Não vale para linhas de grupo nem para a chave `*`. | Nova | ✅ |

## Validação de ativação

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN21 | O parâmetro só vale com `WFU_ATIVO = 1` e, se o `WFU_VLDATV` estiver preenchido, com a expressão verdadeira. Se não, a consulta devolve o `x_Default`. | Nova | ✅ |
| RN22 | Ao salvar, a expressão do `WFU_VLDATV` é testada. Erro, ou retorno que não seja verdadeiro/falso, barra a gravação. | Nova | ✅ |
| RN23 | Erro na expressão durante a consulta devolve o `x_Default`. A rotina que consultou não é derrubada. | Nova | ✅ |
| RN24 | Erro na expressão durante a consulta mostra uma mensagem de erro interno com o parâmetro, o campo, o comando executado e o erro, orientando a contatar um administrador. Sem tela (job), a mensagem vai para o `console.log`. | Nova | ✅ |

## Modo de edição

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN25 | Modo Admin (`0`): só administrador altera e exclui. | Nova | ✅ |
| RN26 | Edição Livre (`1`): todos alteram e excluem, respeitando as demais regras. | Nova | ✅ |
| RN27 | Na inclusão, o modo nasce como Edição Livre e só administrador pode mudá-lo. | Nova | ✅ |
| RN28 | Usuário comum não altera `WFU_VLDITE`, `WFU_VLDATV` e `WFU_MODEDI` depois da inclusão. Administrador sempre pode. | Nova | ✅ |

## Grupos (`WFV_IDPAR`)

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN29 | Uma linha com o `WFV_IDPAR` preenchido inclui na lista todos os itens do parâmetro informado (o grupo). | Nova | ✅ |
| RN30 | O grupo serve só para saber se a chave **faz parte** da lista: o conteúdo do item dentro do grupo não é lido. Chave encontrada pelo grupo devolve o `WFU_CONTEU` do parâmetro consultado, e o `ValidEntityParamList` devolve `.T.`. | Nova | ✅ |
| RN31 | Cada linha tem a chave **ou** o grupo, nunca os dois e nunca nenhum. | Nova | ✅ |
| RN32 | Linha de grupo: o conteúdo fica vazio, o grupo precisa existir e não pode ser o próprio parâmetro. | Nova | ✅ |
| RN33 | Sem grupo dentro de grupo: um grupo não pode ter linhas de grupo, e um parâmetro com linhas de grupo não pode virar grupo. | Nova | ✅ |
| RN34 | Não é permitido incluir como grupo um parâmetro inativo (mensagem descritiva). | Nova | ✅ |
| RN35 | Não é permitido excluir um parâmetro usado como grupo. | Nova | ✅ |
| RN36 | Não é permitido inativar um parâmetro usado como grupo. | Nova | ✅ |
| RN37 | O mesmo grupo não aparece duas vezes na lista: a linha única é `WFV_CHBUSC + WFV_IDPAR`. | Nova | ✅ |
| RN38 | A linha de grupo mostra o apelido do grupo (campo virtual `WFV_DESGRP`, só leitura), preenchido ao informar o grupo e ao abrir a tela. | Nova | ✅ |

**Exemplo:** 7 usuários com permissão em 10 armazéns. Na SX6 seriam 70 parâmetros. Aqui são 2 cadastros: um parâmetro de grupo com os 7 usuários e um parâmetro de permissão com os 10 armazéns no conteúdo padrão e uma linha de grupo apontando para o primeiro.

**Por que um campo próprio (`WFV_IDPAR`) e não um prefixo na chave (ex.: `@USUDEPTO`):** o prefixo foi proposto e recusado pelo cliente por ser um "macete" escondido na chave. O campo próprio deixa explícito o que é chave e o que é grupo, permite consulta padrão (F3) e validações específicas, e mostra o apelido do grupo na linha (`WFV_DESGRP`).

## Segurança

| ID | Regra | Origem | Situação |
|---|---|---|---|
| RN39 | Toda expressão (macro) roda protegida por `try/catch`, em uma rotina única (`ExpressionRunner:Run`, usada pelo cadastro e pela consulta). Um erro vira mensagem ou log com a descrição real do erro e nunca derruba a rotina. Expressão que devolve `Nil` também é tratada como erro. | Nova | ✅ |
