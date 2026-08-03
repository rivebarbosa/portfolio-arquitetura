# ADR-0001: Decompor o sistema em microsserviços por fronteira de negócio, não em monólito

**Status:** aceito
**Data:** 2026-01-15
**Decisores:** Arquitetura, Engenharia

## Contexto

O sistema precisa suportar times separados evoluindo o fluxo de **autorização de pagamento** (alta criticidade, baixa mudança) e o fluxo de **antifraude/disputas** (regras de negócio mudam com frequência, conforme aprendizado sobre fraude) de forma independente, sem que um deploy de um afete o SLA do outro. Além disso, RNF de disponibilidade exige 99,9% no checkout mas apenas 99,5% no split — sinal de que são domínios com perfis operacionais diferentes.

## Alternativas consideradas

### Opção A — Monólito modular
- Prós: menor complexidade operacional inicial, sem necessidade de infraestrutura de mensageria desde o dia 1, mais rápido para o prazo de 4 meses.
- Contras: um bug ou pico de carga no módulo de antifraude pode degradar o checkout (mesmo processo, mesmo runtime); deploys acoplados forçam times a coordenar releases.

### Opção B — Microsserviços por fronteira de negócio (checkout/autorização, split/liquidação, antifraude, disputas)
- Prós: isolamento de falhas entre caminho crítico (checkout) e caminho tolerante a atraso (split); escalonamento independente (RNF de escalabilidade exige 50x só no caminho de autorização); alinha com RNFs de disponibilidade diferentes por domínio.
- Contras: complexidade operacional maior (deploy, observabilidade distribuída, mensageria); exige investimento em idempotência e consistência eventual desde o início.

## Decisão

Decompor por fronteira de negócio em 4 serviços: **checkout/autorização**, **split/liquidação**, **antifraude** e **disputas**, comunicando-se via eventos assíncronos (ver [ADR-0003](0003-processamento-assincrono-do-split.md)) para tudo que não seja o caminho de autorização.

## Padrão arquitetural

**Decompose by Business Capability** (catalogado por Chris Richardson em microservices.io): cada serviço mapeia diretamente a uma capacidade de negócio (autorizar pagamento, dividir valores, avaliar risco, mediar disputa), não a uma camada técnica. Cada serviço corresponde a um **Bounded Context** (DDD — Eric Evans): checkout, split, antifraude e disputas têm modelos de dados e linguagem ubíqua próprios, e só se comunicam através de eventos (contratos explícitos), nunca compartilhando modelo interno diretamente.

## Justificativa

O RNF de disponibilidade (99,9% checkout vs. 99,5% split) e o RNF de escalabilidade (50x apenas no caminho de autorização) já indicavam, antes de qualquer preferência estilística por microsserviços, que esses dois domínios têm perfis operacionais incompatíveis para viverem no mesmo processo. Um monólito exigiria escalar o antifraude e as disputas junto com o pico de Black Friday, desperdiçando recursos sem necessidade.

## Consequências

- Precisamos de um barramento de eventos (Kafka) desde a v1 — não é possível adiar essa decisão de infraestrutura para depois.
- Cada serviço precisa de sua própria estratégia de idempotência e de observabilidade distribuída (trace correlacionado) — custo aceito conscientemente em troca do isolamento.
- O time de engenharia (pequeno, considerando o prazo de 4 meses) precisa operar 4 serviços + mensageria em vez de 1 deployable — risco assumido e mitigado limitando a decomposição a 4 serviços (não 15), evitando explosão de microsserviços prematura.

## Revisitar quando

Se o time de engenharia permanecer com menos de 4 pessoas por mais de 6 meses, o custo operacional de rodar múltiplos serviços pode superar o benefício de isolamento — reavaliar consolidação de antifraude + disputas em um único serviço (ambos de baixa criticidade de latência).
