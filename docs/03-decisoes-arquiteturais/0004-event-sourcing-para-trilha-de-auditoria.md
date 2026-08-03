# ADR-0004: Registrar o histórico de estados de cada transação como eventos imutáveis (event sourcing parcial)

**Status:** aceito
**Data:** 2026-01-28
**Decisores:** Arquitetura, Compliance

## Contexto

Compliance apontou, como restrição inegociável (não uma preferência), que cada transação precisa ter uma trilha auditável e imutável de todos os estados pelos quais passou, retida por no mínimo 5 anos (ver [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#o-que-a-plataforma-é-obrigada-a-guardar-por-quanto-tempo-por-lei)). Além disso, RF-05 exige que compradores, vendedores e atendimento consigam consultar a linha do tempo completa de qualquer transação, não apenas o estado atual.

## Alternativas consideradas

### Opção A — Tabela de transações com campo `status`, atualizado in-place a cada mudança
- Prós: simples, familiar, consultas de "estado atual" são triviais.
- Contras: o histórico de como o estado mudou se perde (um `UPDATE` sobrescreve o valor anterior); para satisfazer a exigência de auditoria, precisaria de uma tabela de log paralela mantida manualmente — com risco real de divergência entre as duas.

### Opção B — Cada transação de pagamento é uma sequência de eventos imutáveis (`TransacaoCriada`, `PagamentoAutorizado`, `SplitCalculado`, `DisputaAberta`, `Liquidada`...), e o estado atual é derivado replay desses eventos
- Prós: a trilha de auditoria é o próprio modelo de dados (não um subproduto que pode divergir); satisfaz a exigência de imutabilidade de forma estrutural, não por convenção; RF-05 (linha do tempo) vem "de graça" do modelo.
- Contras: consultar "o estado atual" exige projeção (materializar uma view a partir dos eventos) em vez de um simples `SELECT`; exige disciplina de nunca apagar ou alterar eventos já publicados, mesmo em caso de bug (correções viram novos eventos compensatórios).

## Decisão

Adotar event sourcing para o histórico de estado de cada transação: os eventos são a fonte da verdade e são append-only; uma projeção (tabela de leitura otimizada) é mantida à parte para consultas rápidas de "estado atual" (RF-05), reconstruível a qualquer momento a partir dos eventos.

## Padrão arquitetural

**Event Sourcing** (Greg Young / Martin Fowler): o estado não é armazenado diretamente, é derivado de uma sequência imutável e append-only de eventos — a trilha de auditoria deixa de ser um subproduto e passa a ser o próprio modelo de dados. A separação entre os eventos (modelo de escrita) e a projeção de leitura otimizada para consulta (RF-05) é uma instância de **CQRS** (Command Query Responsibility Segregation): escrita e leitura têm modelos distintos, otimizados para propósitos diferentes, conectados por uma projeção reconstruível.

## Justificativa

Como a exigência de auditoria não é uma preferência técnica e sim uma restrição regulatória (poder de veto do Compliance, ver [stakeholders.md](../01-discovery/stakeholders.md)), a Opção A exigiria de qualquer forma construir uma trilha de eventos paralela para não violar a restrição — a diferença é que, na Opção A, essa trilha corre o risco de divergir do status "oficial" da tabela principal. Tornar o evento a própria fonte da verdade elimina essa divergência por construção.

## Consequências

- Toda consulta de "estado atual" (usada por RF-05 e pelo suporte ao cliente) depende de uma projeção que precisa ser mantida atualizada — exige investimento em um mecanismo de projeção confiável (idealmente idempotente e reprocessável).
- Bugs não podem ser corrigidos com um `UPDATE` direto no histórico — toda correção precisa de um evento compensatório explícito, o que é mais trabalhoso no curto prazo, mas é exatamente o que a auditoria exige.
- A equipe precisa de familiaridade com o padrão (versionamento de eventos, migração de schema de eventos ao longo do tempo) — curva de aprendizado aceita conscientemente em troca de conformidade regulatória estrutural.

## Revisitar quando

Esta decisão está diretamente amarrada a uma exigência regulatória — só deve ser revisitada se a natureza jurídica do negócio mudar (ex: deixar de operar como intermediador de pagamentos) a ponto de a obrigação de trilha de auditoria de 5 anos deixar de existir.
