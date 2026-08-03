# Portfólio de Arquitetura de Software

Este repositório documenta, do início ao fim, o processo de arquitetura de um sistema fictício: **PagFácil**, uma plataforma de pagamentos para marketplaces (processamento de checkout, split de pagamento entre vendedores, estornos e antifraude).

O objetivo não é o código em si, mas **tornar visível o raciocínio por trás das decisões de arquitetura** — do discovery até os trade-offs técnicos assumidos conscientemente.

> 💡 Este é um projeto fictício, criado para fins de portfólio. O domínio (pagamentos/marketplace) foi escolhido por reunir desafios interessantes de consistência distribuída, compliance e escala.

## Sobre mim

*(Substitua esta seção: 2-3 linhas sobre sua experiência, foco atual — ex: "Arquiteto de soluções com N anos de experiência em sistemas distribuídos, atualmente focado em...")*

- LinkedIn: *seu link*
- E-mail: *seu contato*

## Como navegar neste portfólio

O processo está documentado na ordem em que ele realmente acontece:

| Etapa | Documento | O que você vai encontrar |
|---|---|---|
| 1. Discovery | [`docs/01-discovery/`](docs/01-discovery/README.md) | Contexto de negócio, stakeholders, perguntas feitas e premissas assumidas |
| 2. Requisitos | [`docs/02-requisitos/`](docs/02-requisitos/requisitos-funcionais.md) | Requisitos funcionais e não funcionais, com critérios de aceite mensuráveis |
| 3. Decisões | [`docs/03-decisoes-arquiteturais/`](docs/03-decisoes-arquiteturais/) | ADRs (Architecture Decision Records) — cada decisão relevante, com alternativas consideradas e motivo da escolha |
| 4. Diagramas | [`docs/04-diagramas/`](docs/04-diagramas/README.md) | Modelo C4 (contexto, containers, componentes) e diagrama de sequência do fluxo crítico |
| 5. Trade-offs | [`docs/05-trade-offs/analise-trade-offs.md`](docs/05-trade-offs/analise-trade-offs.md) | Análise explícita do que foi ganho e perdido em cada decisão relevante |

## Resumo do sistema

PagFácil processa pagamentos de um marketplace multi-vendedor: o comprador paga um valor único no checkout, e a plataforma distribui (*split*) esse valor entre os vendedores envolvidos e a própria plataforma (comissão), com janela de disputa/estorno e verificação antifraude antes da liquidação.

**Principais desafios de arquitetura abordados:**
- Consistência eventual em transações financeiras distribuídas (padrão Saga)
- Escala para picos sazonais (Black Friday) sem degradar SLA de checkout
- Auditabilidade e conformidade regulatória (rastreabilidade total de cada centavo)
- Idempotência em um domínio onde duplicar uma cobrança é inaceitável

## Stack de exemplo

*(Ajuste para a stack que você realmente domina/quer demonstrar)*

- Backend: microsserviços (linguagem a definir por você)
- Mensageria: Kafka (event-driven, para o fluxo de split e liquidação)
- Bancos de dados: PostgreSQL (dados transacionais) + Redis (idempotência/cache)
- Observabilidade: OpenTelemetry + Grafana

---

*Portfólio criado para demonstrar processo de arquitetura de software. Não representa um sistema em produção.*
