# Diagrama de Contexto (C4 — Nível 1)

Mostra o sistema PagFácil como uma caixa única e seus relacionamentos com atores e sistemas externos. Não há detalhe interno aqui de propósito — o objetivo é alinhar com stakeholders de negócio sobre "quem fala com o quê".

```mermaid
C4Context
    title Diagrama de Contexto — PagFácil

    Person(comprador, "Comprador", "Compra produtos no marketplace")
    Person(vendedor, "Vendedor", "Vende produtos, recebe via split")
    Person(atendimento, "Atendimento", "Consulta status e media disputas")

    System(pagfacil, "PagFácil", "Processa checkout, split, disputas e liquidação")

    System_Ext(adquirente, "Adquirente de Pagamento", "Autoriza cartão/Pix/boleto")
    System_Ext(bacen, "Banco Central (SPI/Open Finance)", "Regras de intermediador de pagamento")
    System_Ext(marketplace, "Vitrine (Marketplace)", "Catálogo, carrinho, pedidos")

    Rel(comprador, marketplace, "Monta carrinho")
    Rel(marketplace, pagfacil, "Envia carrinho para checkout")
    Rel(comprador, pagfacil, "Paga (cartão/Pix/boleto)")
    Rel(pagfacil, adquirente, "Solicita autorização")
    Rel(pagfacil, vendedor, "Notifica venda aprovada / paga via split")
    Rel(pagfacil, bacen, "Reporta transações (compliance)")
    Rel(atendimento, pagfacil, "Consulta status / media disputa")
```

**Leitura:** o marketplace (Vitrine) permanece responsável pelo catálogo e carrinho — o PagFácil entra apenas a partir do checkout. Essa fronteira foi definida no discovery ([contexto-negocio.md](../01-discovery/contexto-negocio.md)) e evita que o sistema de pagamentos vire, sem necessidade, dono de dados de catálogo.
