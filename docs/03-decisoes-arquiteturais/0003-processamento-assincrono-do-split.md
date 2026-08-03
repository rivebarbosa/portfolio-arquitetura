# ADR-0003: Separar autorização (síncrona) de split/liquidação (assíncrono)

**Status:** aceito
**Data:** 2026-01-22
**Decisores:** Arquitetura

## Contexto

O RNF de desempenho exige P95 < 3s para a autorização do pagamento (caminho que o comprador espera ver na tela). Já a resposta de discovery sobre expectativa do vendedor mostrou que ele tolera até 5 minutos para saber que vai receber (ver [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#quanto-tempo-o-vendedor-pode-esperar-para-saber-que-vai-receber)). Calcular o split (quanto cada vendedor recebe, quanto é comissão) e verificar antifraude antes de confirmar tudo em uma única resposta síncrona colocaria essas etapas extras no orçamento de latência de 3s do comprador, sem necessidade real.

## Alternativas consideradas

### Opção A — Tudo síncrono: autorizar, calcular split e checar antifraude na mesma requisição de checkout
- Prós: mais simples de raciocinar (um único fluxo linear); resultado final já sai pronto.
- Contras: soma a latência de 3 operações no orçamento de 3s do checkout; qualquer lentidão no serviço de antifraude derruba a conversão do checkout inteiro; acopla a disponibilidade do checkout (99,9% exigido) à do antifraude (que não precisa desse SLA).

### Opção B — Autorização síncrona; split, antifraude e liquidação processados de forma assíncrona via eventos
- Prós: o comprador só espera pela autorização (que é o único passo com SLA de 3s); split e antifraude escalam e falham de forma independente do checkout; alinhado ao RNF de disponibilidade diferenciado por domínio.
- Contras: o vendedor não sabe o valor exato "na hora" (só minutos depois); exige comunicar ao usuário que o valor "está em processamento", o que é uma decisão de produto, não só técnica.

## Decisão

A autorização do pagamento é síncrona e retorna ao comprador em até 3s (P95). O cálculo de split, a verificação antifraude e a liquidação acontecem de forma assíncrona, disparados por um evento `PagamentoAutorizado`, com conclusão em até 5 minutos (RNF).

## Justificativa

Essa é a tradução direta de uma resposta de discovery em arquitetura: a tolerância de minutos do vendedor não é um detalhe de UX, é o que permite desacoplar dois RNFs de latência incompatíveis (3s vs. minutos) sem forçar o caminho mais lento a definir o SLA do caminho mais rápido.

## Consequências

- O sistema precisa comunicar ao vendedor um estado intermediário ("venda aprovada, valor em processamento") em vez de um valor final imediato — exige alinhamento com produto sobre a UX dessa espera.
- Introduz a necessidade de idempotência no consumo do evento `PagamentoAutorizado` (reprocessamento de mensagens não pode duplicar o split).
- Cria dependência do padrão Saga (ver [ADR-0002](0002-padrao-saga-para-consistencia-transacional.md)) para garantir que uma falha no split assíncrono não deixe a transação em estado inconsistente.

## Revisitar quando

Se o negócio decidir oferecer split "instantâneo" como diferencial competitivo no futuro, essa decisão precisa ser reaberta — provavelmente exigindo otimizar o antifraude para rodar em orçamento de latência muito menor, ou aceitar liberar o split antes da checagem completa (com maior risco).
