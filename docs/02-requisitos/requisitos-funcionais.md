# Requisitos Funcionais

Cada requisito abaixo nasceu de uma resposta específica do discovery (ver [`01-discovery/`](../01-discovery/)) — nenhum foi assumido por padrão de mercado sem checar se se aplicava a este negócio.

## RF-01 — Checkout unificado multi-vendedor

O comprador deve conseguir pagar, em uma única transação, por produtos de vendedores diferentes no mesmo carrinho.

- **Critério de aceite:** um carrinho com N vendedores gera 1 única cobrança ao comprador e N registros de valor a receber (um por vendedor), com a soma dos N batendo exatamente com o valor cobrado menos a comissão da plataforma.
- **Origem:** [contexto-negocio.md](../01-discovery/contexto-negocio.md#o-problema)

## RF-02 — Split automático de valores

Após a aprovação do pagamento, o sistema deve dividir o valor entre os vendedores envolvidos e reter automaticamente a comissão da plataforma, sem intervenção manual.

- **Critério de aceite:** para cada transação aprovada, existe um evento de split gerado em até 5 minutos, contendo o valor líquido por vendedor e o valor de comissão retido.
- **Origem:** [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#o-que-acontece-se-o-split-falhar-depois-que-o-comprador-já-foi-cobrado)

## RF-03 — Janela de disputa antes da liquidação

O valor de cada vendedor fica retido por um período (D+7) antes de ser liberado, durante o qual o comprador pode abrir uma disputa.

- **Critério de aceite:** nenhum valor é transferido para a conta do vendedor antes de D+7 corridos a partir da aprovação, salvo se o próprio vendedor solicitar antecipação (fora de escopo desta v1).
- **Origem:** [contexto-negocio.md](../01-discovery/contexto-negocio.md#objetivo-de-negócio)

## RF-04 — Abertura e mediação de disputas

O comprador pode abrir uma disputa sobre uma transação específica; a plataforma media o processo e decide se o valor retido é liberado ao vendedor, estornado ao comprador, ou parcialmente dividido.

- **Critério de aceite:** uma disputa aberta congela a liquidação daquela transação especificamente (não afeta outras transações do mesmo vendedor); toda decisão de disputa gera um evento auditável com o motivo.
- **Origem:** [stakeholders.md](../01-discovery/stakeholders.md)

## RF-05 — Consulta de status da transação

Compradores, vendedores e atendimento devem conseguir consultar o status atual e o histórico de estados de qualquer transação.

- **Critério de aceite:** dado um ID de transação, o sistema retorna o estado atual e a linha do tempo completa de eventos (criada → autorizada → split calculado → liquidada/disputada), sem exigir acesso direto ao banco de dados.
- **Origem:** [stakeholders.md](../01-discovery/stakeholders.md) (Time de Atendimento)

## RF-06 — Verificação antifraude pré-liquidação

Toda transação passa por uma verificação antifraude antes de ser liberada para liquidação; transações sinalizadas entram em revisão manual.

- **Critério de aceite:** transações com score de risco acima do limiar configurado não avançam automaticamente para liquidação — ficam em fila de revisão com SLA de resposta de 24h.
- **Origem:** requisito derivado da obrigação regulatória de compliance (ver [requisitos não funcionais — conformidade](requisitos-nao-funcionais.md#conformidade-regulatória))

## RF-07 — Onboarding e KYC de vendedores

Um vendedor só pode receber valores via split depois de completar um processo de verificação de identidade (KYC).

- **Critério de aceite:** transações destinadas a um vendedor sem KYC aprovado ficam retidas (não recusadas) até a aprovação, com notificação ao vendedor sobre a pendência.
- **Origem:** [stakeholders.md](../01-discovery/stakeholders.md) (Compliance/Jurídico)

---

**Fora de escopo explícito** (ver [contexto-negocio.md](../01-discovery/contexto-negocio.md#fora-de-escopo-decidido-no-discovery-não-depois)): pagamento em moeda estrangeira, split em múltiplos níveis (sub-afiliados), emissão de nota fiscal.
