# ADR-0002: Usar padrão Saga com compensação em vez de transação distribuída (2PC)

**Status:** aceito
**Data:** 2026-01-20
**Decisores:** Arquitetura

## Contexto

Uma única operação de negócio ("comprador paga → vendedores recebem") atravessa múltiplos serviços (autorização, split, KYC, antifraude), cada um com seu próprio banco de dados. O discovery deixou claro que não pode existir um estado final em que o comprador é cobrado e nenhum vendedor recebe, nem o inverso (ver [perguntas-discovery.md](../01-discovery/perguntas-discovery.md#o-que-acontece-se-o-split-falhar-depois-que-o-comprador-já-foi-cobrado)) — mas também que o resultado final pode levar minutos para se consolidar (o vendedor tolera espera).

## Alternativas consideradas

### Opção A — Two-Phase Commit (2PC) entre os serviços
- Prós: consistência forte, sem estados intermediários visíveis.
- Contras: exige que todos os serviços participem de um coordenador de transação distribuída, bloqueando recursos durante a janela de commit; não escala bem sob o pico de 50x exigido pelo RNF de escalabilidade; forte acoplamento entre serviços que o ADR-0001 justamente busca desacoplar.

### Opção B — Saga coreografada, com eventos de compensação para cada passo que pode falhar
- Prós: cada serviço processa sua parte de forma independente e escalável; falhas são tratadas com ações de compensação explícitas (ex: estornar autorização se o KYC do vendedor falhar); compatível com a tolerância a atraso de minutos identificada no discovery.
- Contras: consistência é eventual, não instantânea — existe uma janela onde o estado está "em andamento"; exige desenhar explicitamente a ação de compensação para cada passo, aumentando a superfície de casos de teste.

## Decisão

Implementar o fluxo de pagamento como uma **Saga coreografada**: cada serviço publica um evento ao concluir sua etapa, e cada etapa subsequente tem uma ação de compensação definida caso um passo posterior falhe (ex: se o antifraude sinalizar depois que o split já foi calculado, o evento de compensação reverte o split antes da liquidação).

## Justificativa

A resposta do discovery de que o resultado final precisa ser tudo-ou-nada, mas pode levar minutos, é exatamente o cenário para o qual Saga foi desenhado — e o 2PC introduziria bloqueio síncrono entre serviços que o RNF de escalabilidade (50x no pico) não suporta bem.

## Consequências

- Cada serviço precisa expor uma operação de compensação para cada operação que participa da saga — isso é trabalho de design adicional que precisa ser feito por serviço, não é "grátis".
- Existe uma janela de tempo em que a transação está em estado intermediário e visível (ex: "autorizada, aguardando split") — isso é aceito porque RF-05 (consulta de status) já previa expor esse histórico de estados ao usuário.
- Requer um mecanismo de detecção de saga "travada" (timeout) para disparar compensação automaticamente se uma etapa não responder dentro do esperado.

## Revisitar quando

Se o número de passos da saga crescer muito (ex: novos parceiros de pagamento, novos tipos de verificação), considerar migrar de coreografia para orquestração centralizada (um orquestrador explícito), pois a coreografia fica difícil de auditar visualmente a partir de ~6-7 passos.
