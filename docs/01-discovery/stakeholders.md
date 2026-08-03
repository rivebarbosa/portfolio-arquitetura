# Mapa de Stakeholders

| Stakeholder | O que precisa do sistema | Nível de influência na arquitetura |
|---|---|---|
| **Compradores** | Checkout rápido, único, com meios de pagamento variados (cartão, Pix, boleto) | Alto — define os requisitos de UX/latência do fluxo crítico |
| **Vendedores** | Previsibilidade de quando recebem, visibilidade de taxas e disputas | Alto — define regras de liquidação e janela de disputa |
| **Time Financeiro (plataforma)** | Retenção automática de comissão, conciliação contábil, relatórios auditáveis | Alto — define requisitos de auditabilidade e consistência |
| **Compliance/Jurídico** | Aderência às regras do Banco Central para intermediadores de pagamento, KYC de vendedores | Alto — define restrições inegociáveis (não são "requisitos", são limites) |
| **Time de Atendimento** | Visibilidade do status de cada transação para responder ao cliente | Médio — define requisitos de observabilidade/consulta |
| **Board/Diretoria** | Prazo de 4 meses, custo operacional controlado | Médio — influencia decisões de build vs. buy |

## Por que este mapa importa para a arquitetura

Dois pontos deste mapa mudaram decisões técnicas diretamente:

1. **Compliance tem poder de veto, não de sugestão.** Isso significa que requisitos como "toda transação deve ser rastreável e imutável" não são negociáveis por performance — viraram restrição arquitetural (ver [ADR-0004](../03-decisoes-arquiteturais/0004-event-sourcing-para-trilha-de-auditoria.md)).

2. **Vendedores e compradores têm expectativas de latência diferentes.** O comprador precisa de resposta em segundos (autorização do pagamento); o vendedor pode esperar minutos/horas pela confirmação do split. Essa diferença foi o que justificou separar o fluxo em **autorização síncrona** + **liquidação assíncrona** (ver [ADR-0003](../03-decisoes-arquiteturais/0003-processamento-assincrono-do-split.md)), em vez de tentar fazer tudo em uma única transação síncrona.
