# Contexto de Negócio

## O problema

O marketplace "Vitrine" conecta compradores a milhares de pequenos vendedores. Hoje, cada vendedor processa seus próprios pagamentos via link de pagamento externo, o que gera três dores centrais:

- **Experiência de checkout fragmentada**: um carrinho com produtos de 3 vendedores diferentes exige 3 pagamentos separados.
- **Falta de confiança do comprador**: sem intermediação, disputas e estornos são negociados diretamente com o vendedor, sem garantia da plataforma.
- **Falta de visibilidade financeira da plataforma**: a comissão é cobrada manualmente por boleto mensal, gerando inadimplência e atraso de caixa.

## Objetivo de negócio

Centralizar o pagamento na própria plataforma: o comprador paga **uma única vez**, e a plataforma se responsabiliza por dividir o valor entre os vendedores envolvidos (split), reter a comissão automaticamente, e gerenciar o ciclo de disputa/estorno antes da liquidação para o vendedor.

## Métricas de sucesso (o que define "funcionou")

| Métrica | Situação atual | Meta pós-lançamento |
|---|---|---|
| Checkout com múltiplos vendedores | Impossível (requer N pagamentos) | 1 pagamento único |
| Tempo médio de checkout | N/A (fora da plataforma) | < 3s para autorização |
| Inadimplência de comissão | ~18% dos vendedores atrasam | Comissão retida automaticamente (0% de inadimplência) |
| Disputas resolvidas pela plataforma | 0% (negociação direta) | 100% mediadas, com fundo retido durante a janela de disputa |
| Disponibilidade do checkout | N/A | 99,9% (não pode cair em datas de pico) |

## Restrições conhecidas desde o início

- **Regulatória**: o sistema vai operar como *intermediador de pagamentos*, o que impõe obrigações do Banco Central (rastreabilidade de cada transação, prazos de liquidação, KYC = Know Your Customer ("conheça seu cliente") KYC dos vendedores).
- **Prazo**: o board quer uma primeira versão operacional em 4 meses, para rodar em paralelo ao modelo antigo antes de descontinuá-lo.
- **Sazonalidade**: o pico de Black Friday já é conhecido (~35x o tráfego médio) e não é negociável — o sistema precisa suportar esse pico desde o dia 1, mesmo que o volume médio inicial seja baixo.

## Fora de escopo (decidido no discovery, não depois)

- Pagamentos internacionais (moeda estrangeira) — fica para uma fase 2.
- Split para mais de 1 nível (ex: sub-vendedores/afiliados) — não existe hoje no modelo de negócio.
- Emissão de nota fiscal — permanece responsabilidade do vendedor, fora do sistema.

Definir o que está **fora** de escopo neste momento evitou que requisitos não funcionais (como suporte a múltiplas moedas) inflassem a arquitetura sem necessidade real.
