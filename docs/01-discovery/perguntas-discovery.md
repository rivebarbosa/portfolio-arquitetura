# Perguntas de Discovery

Esta é a parte mais subestimada do processo: a qualidade da arquitetura depende diretamente da qualidade das perguntas feitas *antes* de desenhar qualquer coisa. Abaixo, as perguntas de maior impacto, por que foram feitas, e o que a resposta (ou a falta dela) mudou no desenho.

## "O comprador pode pagar por produtos de vendedores diferentes em um único checkout?"

**Resposta:** Sim, esse é o requisito central do negócio.
**Impacto:** Isso sozinho elimina qualquer solução simples de "repassar o pagamento para o vendedor" — o sistema precisa reter o valor total e fazer o split internamente. Sem essa pergunta, o desenho inicial (mais simples) teria sido descartado tarde demais.

## "O que acontece se o split falhar depois que o comprador já foi cobrado?"

**Resposta:** Não pode existir cenário em que o comprador paga e nenhum vendedor recebe, nem o inverso. A operação tem que ser tudo-ou-nada do ponto de vista do resultado final, mesmo que os passos individuais aconteçam em momentos diferentes.
**Impacto:** Essa resposta é o motivo direto de adotarmos o padrão **Saga com compensação** em vez de uma transação distribuída tradicional (2PC) — ver [ADR-0002](../03-decisoes-arquiteturais/0002-padrao-saga-para-consistencia-transacional.md).

## "Quanto tempo o vendedor pode esperar para saber que vai receber?"

**Resposta:** Até 5 minutos é aceitável; o vendedor só precisa da confirmação instantânea de que a *venda foi aprovada* — o valor em si só é liquidado depois da janela de disputa (D+7).
**Impacto:** Isso abriu espaço para processar o split de forma assíncrona, tirando pressão de latência do caminho crítico do checkout. Sem essa pergunta, poderíamos ter otimizado (sem necessidade) para split síncrono em <1s.

## "Existe um teto conhecido de transações por segundo no pico?"

**Resposta:** Não havia número exato, mas o time de negócio sabia que a Black Friday do ano anterior (no modelo antigo) teve ~35x o tráfego médio, com pico de alguns minutos.
**Impacto:** Como não havia número preciso, assumimos a premissa conservadora de dimensionar para 50x o tráfego médio nos componentes do caminho crítico (autorização), documentada explicitamente como premissa (não como fato) — ver [requisitos não funcionais](../02-requisitos/requisitos-nao-funcionais.md#escalabilidade).

## "O que a plataforma é obrigada a guardar, por quanto tempo, por lei?"

**Resposta:** Compliance apontou que, como intermediador de pagamento, cada transação precisa ser rastreável e auditável por no mínimo 5 anos, com trilha imutável de todos os estados pelos quais ela passou.
**Impacto:** Essa foi a pergunta que mais restringiu o desenho de dados — motivou registrar o histórico de estados como **eventos imutáveis** em vez de apenas atualizar o status em uma tabela (ver [ADR-0004](../03-decisoes-arquiteturais/0004-event-sourcing-para-trilha-de-auditoria.md)).

## Premissas assumidas (perguntas sem resposta definitiva no discovery)

Quando uma resposta não estava disponível, a premissa foi documentada explicitamente — para poder ser revisitada depois, em vez de virar um "fato" escondido no código:

- **Premissa:** o volume de disputas/estornos será baixo (<2% das transações) nos primeiros 6 meses. *Se essa premissa se mostrar falsa, a arquitetura da fila de disputas precisa ser revisitada (hoje dimensionada para baixo volume).*
- **Premissa:** todos os vendedores aceitarão liquidação em D+7 fixo. *Se no futuro houver necessidade de prazos diferenciados por vendedor, o modelo de dados de liquidação precisa ser generalizado (hoje assume prazo único).*
