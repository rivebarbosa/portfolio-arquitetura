# Análise de Trade-offs

Um ADR documenta *uma* decisão isolada. Este documento existe para explicitar os trade-offs **cruzados** — o que ganhamos e o que conscientemente abrimos mão, olhando o desenho como um todo, não decisão por decisão.

## Consistência forte vs. disponibilidade/escala (CAP, na prática)

**O que escolhemos:** consistência eventual (Saga + event sourcing) em vez de transações distribuídas fortes (2PC).

**O que ganhamos:** os serviços de split, antifraude e disputas escalam e falham de forma independente; o pico de 50x (RNF de escalabilidade) não é limitado pelo serviço mais lento da cadeia.

**O que perdemos:** existe uma janela de tempo (segundos a minutos) em que a transação está em estado intermediário e visível ao usuário ("aprovada, processando"). Isso é uma escolha de produto tanto quanto técnica — e só foi possível porque o discovery confirmou explicitamente que o vendedor tolera essa espera. Se essa tolerância não existisse, esse trade-off inteiro precisaria ser reaberto.

## Simplicidade operacional vs. isolamento de falhas

**O que escolhemos:** 4 microsserviços (ADR-0001) em vez de um monólito modular.

**O que ganhamos:** um pico de carga ou bug no serviço de antifraude não derruba o checkout; cada serviço escala conforme seu próprio perfil de demanda.

**O que perdemos:** complexidade operacional real — mensageria, observabilidade distribuída, mais superfícies de deploy — para um time que tem apenas 4 meses de prazo. Esse é o trade-off mais arriscado do projeto: a aposta é que o custo de operar 4 serviços é menor que o custo de um incidente de checkout causado por acoplamento com antifraude. Está documentado como reversível (ver "Revisitar quando" do [ADR-0001](../03-decisoes-arquiteturais/0001-microsservicos-por-fronteira-de-negocio.md)) exatamente porque é uma aposta, não uma certeza.

## Auditabilidade estrutural vs. simplicidade de consulta

**O que escolhemos:** event sourcing (ADR-0004) em vez de uma tabela de status simples.

**O que ganhamos:** a trilha de auditoria exigida por Compliance é uma propriedade estrutural do sistema, não uma tabela de log que pode divergir do estado "oficial".

**O que perdemos:** toda consulta de "estado atual" depende de uma projeção derivada — mais uma peça de infraestrutura para manter correta e atualizada, e mais uma fonte potencial de bugs sutis (projeção desatualizada) que não existiria com um simples `UPDATE`.

## Build vs. buy: por que não usar um gateway de pagamento completo (ex: Stripe Connect) pronto?

Esta pergunta foi levantada no discovery mas está fora do escopo técnico deste portfólio — vale registrar o raciocínio, mesmo assim:

- **A favor de comprar:** soluções como Stripe Connect já resolvem split, KYC e compliance PCI, reduzindo o prazo de 4 meses drasticamente.
- **Contra:** o modelo de negócio exige regras específicas de disputa e janela D+7 que nem sempre são configuráveis em soluções prontas; e depender de um único fornecedor externo para a operação financeira central do negócio é um risco estratégico que o board preferiu não assumir nesta fase.
- **Registrado aqui, não como ADR:** porque é uma decisão de negócio (make vs. buy) que antecede a arquitetura, não uma decisão de arquitetura em si — mas influencia diretamente por que este portfólio assume "construir" como premissa.

## O padrão por trás de todos esses trade-offs

Em nenhum caso a escolha foi "a opção tecnicamente mais sofisticada". Em todos os casos, a escolha foi a que respondia diretamente a uma restrição concreta levantada no discovery ou nos requisitos — a sofisticação (Saga, event sourcing, microsserviços) é consequência dessas restrições, não um objetivo em si.
