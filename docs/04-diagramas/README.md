# Diagramas

Diagramas seguem o **modelo C4** (Contexto → Containers → Componentes), do mais abstrato para o mais detalhado. A ideia é que qualquer pessoa — de um stakeholder de negócio a um novo engenheiro no time — encontre o nível de detalhe certo para sua necessidade, sem precisar entender tudo de uma vez.

Todos os diagramas estão em Mermaid (renderizam nativamente no GitHub, sem precisar de imagem exportada).

1. [`contexto.md`](contexto.md) — o sistema PagFácil e como ele se relaciona com atores externos (comprador, vendedor, adquirente de pagamento, Banco Central)
2. [`containers.md`](containers.md) — os serviços internos definidos no [ADR-0001](../03-decisoes-arquiteturais/0001-microsservicos-por-fronteira-de-negocio.md) e como se comunicam
3. [`sequencia-checkout.md`](sequencia-checkout.md) — o fluxo ponta a ponta de uma compra, desde o checkout até a liquidação, mostrando exatamente onde o síncrono termina e o assíncrono começa (ver [ADR-0003](../03-decisoes-arquiteturais/0003-processamento-assincrono-do-split.md))
