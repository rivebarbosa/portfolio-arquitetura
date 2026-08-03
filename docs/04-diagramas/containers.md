# Diagrama de Containers (C4 — Nível 2)

Detalha os serviços internos do PagFácil, decompostos conforme [ADR-0001](../03-decisoes-arquiteturais/0001-microsservicos-por-fronteira-de-negocio.md), e como se comunicam entre si — síncrono no caminho crítico, assíncrono (via eventos) em todo o resto, conforme [ADR-0003](../03-decisoes-arquiteturais/0003-processamento-assincrono-do-split.md).

```mermaid
C4Container
    title Diagrama de Containers — PagFácil

    Person(comprador, "Comprador")
    Person(vendedor, "Vendedor")

    System_Boundary(pagfacil, "PagFácil") {
        Container(checkout, "Serviço de Checkout/Autorização", "API síncrona", "Recebe carrinho, solicita autorização ao adquirente, P95 < 3s")
        Container(split, "Serviço de Split/Liquidação", "Consumer assíncrono", "Calcula divisão de valores, aplica janela D+7")
        Container(antifraude, "Serviço de Antifraude", "Consumer assíncrono", "Calcula score de risco antes da liquidação")
        Container(disputas, "Serviço de Disputas", "API + Consumer", "Gerencia abertura e mediação de disputas")
        ContainerDb(eventstore, "Event Store", "PostgreSQL (append-only)", "Trilha imutável de eventos por transação (ADR-0004)")
        ContainerDb(projecao, "Projeção de Leitura", "PostgreSQL (view materializada)", "Estado atual otimizado para consulta (RF-05)")
        Container(barramento, "Barramento de Eventos", "Kafka", "Desacopla autorização de split/antifraude/disputas")
    }

    System_Ext(adquirente, "Adquirente de Pagamento")

    Rel(comprador, checkout, "Envia pagamento", "HTTPS")
    Rel(checkout, adquirente, "Autoriza", "HTTPS")
    Rel(checkout, eventstore, "Grava PagamentoAutorizado")
    Rel(checkout, barramento, "Publica evento")
    Rel(barramento, split, "Consome PagamentoAutorizado")
    Rel(barramento, antifraude, "Consome PagamentoAutorizado")
    Rel(split, eventstore, "Grava SplitCalculado")
    Rel(antifraude, barramento, "Publica TransacaoSinalizada (se houver risco)")
    Rel(barramento, disputas, "Consome eventos de disputa")
    Rel(vendedor, disputas, "Consulta/recebe notificação")
    Rel(eventstore, projecao, "Projeta estado atual")
    Rel(vendedor, projecao, "Consulta status (RF-05)")
```

**Leitura:** note que **nenhum serviço fala diretamente com outro** fora do caminho de autorização — tudo passa pelo barramento de eventos. Essa é a materialização direta da Saga coreografada definida no [ADR-0002](../03-decisoes-arquiteturais/0002-padrao-saga-para-consistencia-transacional.md): cada serviço reage a eventos, sem um orquestrador central.
