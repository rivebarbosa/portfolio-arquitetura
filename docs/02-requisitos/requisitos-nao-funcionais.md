# Requisitos Não Funcionais

Requisitos não funcionais só têm valor quando são **mensuráveis**. Cada item abaixo tem um número, e cada número tem uma origem rastreável no discovery — nenhum foi copiado de um "padrão de mercado" genérico sem checar se fazia sentido para este sistema.

## Desempenho

| Requisito | Alvo | Origem |
|---|---|---|
| Latência de autorização do pagamento (P95) | < 3s | Meta de negócio para não perder conversão no checkout ([contexto-negocio.md](../01-discovery/contexto-negocio.md#métricas-de-sucesso-o-que-define-funcionou)) |
| Latência de autorização do pagamento (P99) | < 8s | Mesmo, com margem para timeouts de adquirentes externos |
| Latência do split (do pagamento aprovado até o evento de split calculado) | < 5 min | Resposta de discovery — vendedor tolera minutos, não segundos ([perguntas-discovery.md](../01-discovery/perguntas-discovery.md#quanto-tempo-o-vendedor-pode-esperar-para-saber-que-vai-receber)) |

## Escalabilidade

| Requisito | Alvo | Origem |
|---|---|---|
| Pico sustentado no caminho de autorização | 50x o tráfego médio diário, por pelo menos 30 minutos contínuos | Premissa conservadora sobre o pico real de Black Friday do ano anterior (35x), documentada como **premissa**, não fato — ver [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#existe-um-teto-conhecido-de-transações-por-segundo-no-pico) |
| Escalonamento do processamento de split | Deve escalar horizontalmente de forma independente do caminho de autorização | Consequência direta de terem sido desacoplados (ver [ADR-0003](../03-decisoes-arquiteturais/0003-processamento-assincrono-do-split.md)) |

## Disponibilidade

| Requisito | Alvo | Origem |
|---|---|---|
| Disponibilidade do fluxo de checkout/autorização | 99,9% (≈ 8h43min de indisponibilidade/ano) | [contexto-negocio.md](../01-discovery/contexto-negocio.md#métricas-de-sucesso-o-que-define-funcionou) |
| Disponibilidade do fluxo de split/liquidação | 99,5% — pode tolerar atraso (é assíncrono), mas não pode perder eventos | Consequência de RF-02 combinado com a natureza assíncrona do fluxo |

## Consistência de dados

| Requisito | Alvo | Origem |
|---|---|---|
| Nenhuma transação pode resultar em "comprador cobrado, nenhum vendedor recebe" (ou o inverso) | Consistência eventual garantida por compensação (Saga), não consistência forte instantânea | [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#o-que-acontece-se-o-split-falhar-depois-que-o-comprador-já-foi-cobrado) |
| Idempotência de cobrança | Zero cobranças duplicadas, mesmo sob retry de rede ou reprocessamento de mensagens | Requisito inegociável em um sistema financeiro — nenhuma pergunta específica precisou ser feita, é um limite do domínio |

## Segurança

| Requisito | Alvo | Origem |
|---|---|---|
| Dados de cartão | Nunca armazenados diretamente — tokenização via gateway PCI-DSS compliant | Restrição inegociável de compliance |
| Controle de acesso | Cada papel (comprador, vendedor, atendimento, financeiro) só acessa os dados de transação aos quais tem direito, via autorização por escopo | [stakeholders.md](../01-discovery/stakeholders.md) |

## Conformidade regulatória

| Requisito | Alvo | Origem |
|---|---|---|
| Rastreabilidade de transações | Histórico completo e imutável de estados por transação, retido por no mínimo 5 anos | [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#o-que-a-plataforma-é-obrigada-a-guardar-por-quanto-tempo-por-lei) — motivou [ADR-0004](../03-decisoes-arquiteturais/0004-event-sourcing-para-trilha-de-auditoria.md) |
| KYC de vendedores antes da liquidação | Bloqueio de liquidação (não de venda) até KYC aprovado | RF-07 |

## Observabilidade

| Requisito | Alvo | Origem |
|---|---|---|
| Rastreamento ponta a ponta de uma transação | Um trace único correlaciona autorização → split → liquidação/disputa, mesmo passando por serviços assíncronos diferentes | RF-05 (consulta de status) + necessidade de debug operacional |

---

## Nota sobre premissas não confirmadas

Dois itens desta tabela (o pico de 50x e o volume de disputas) são **premissas**, não fatos confirmados pelo negócio — estão documentados como tal em [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#premissas-assumidas-perguntas-sem-resposta-definitiva-no-discovery) para serem revisitados quando houver dado real.
