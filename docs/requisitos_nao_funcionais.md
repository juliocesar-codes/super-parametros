# Requisitos não funcionais (RNF)

| ID | Requisito | Como é atendido |
|---|---|---|
| RNF01 | **Escalabilidade:** novos parâmetros, chaves e grupos são cadastrados sem alterar o código, e a solução suporta o crescimento de parâmetros, itens e filiais. | Tudo é cadastro (WFU/WFV). Grupos evitam cadastros repetidos. A verificação de uso como grupo é feita por SQL, sem percorrer a tabela. |
| RNF02 | **Nomes descritivos:** campos, variáveis, métodos e mensagens têm nomes que deixam claro o que representam. | Padrão de código em `rules/desenvolvimento_codigo/padrao_codigo_prw_tlpp.md` (variáveis `<tipo>_<Nome>`, Protheus.doc em toda função, mensagens com versão e tipo do erro). |
| RNF03 | **Tela fácil de navegar:** cabeçalho e itens na mesma tela, títulos claros e mensagens que orientam o usuário. | Tela MVC única, apelido do grupo visível na linha, combo do tipo explicando o Lógico (V/F), mensagens com o motivo e a orientação de correção. |
| RNF04 | **Robustez:** uma expressão (macro) cadastrada errada nunca derruba a rotina que consulta o parâmetro. | `try/catch` em uma rotina única (`ExpressionRunner:Run`), com mensagem em tela ou log em job (RN39). |
