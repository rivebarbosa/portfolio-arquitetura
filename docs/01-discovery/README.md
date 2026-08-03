# Discovery

Antes de desenhar qualquer diagrama, o objetivo do discovery é responder três perguntas: **qual problema de negócio estamos resolvendo, para quem, e o que "sucesso" significa em números**.

Esta pasta documenta esse processo em três documentos:

1. [`contexto-negocio.md`](contexto-negocio.md) — o problema, objetivos de negócio e métricas de sucesso
2. [`stakeholders.md`](stakeholders.md) — mapa de stakeholders e o que cada um precisava do sistema
3. [`perguntas-discovery.md`](perguntas-discovery.md) — as perguntas efetivamente feitas, respostas obtidas e premissas assumidas onde não havia resposta

## Metodologia usada

- **Entrevistas estruturadas** com representantes de cada stakeholder (aqui, simuladas para fins de portfólio, mas seguindo um roteiro real de perguntas de arquitetura: fluxos críticos, picos de uso, restrições regulatórias, integrações existentes).
- **Event Storming leve** para mapear o fluxo de checkout ponta a ponta antes de desenhar qualquer componente técnico.
- **Priorização por impacto x incerteza**: perguntas com alto impacto na arquitetura e alta incerteza foram resolvidas antes de qualquer linha de requisito ser escrita (ex: "o split acontece em tempo real ou pode ser assíncrono?").

O critério para sair do discovery e entrar em requisitos foi ter clareza sobre: os fluxos críticos do negócio, os limites regulatórios inegociáveis, e as restrições de escala conhecidas (mesmo que aproximadas).
