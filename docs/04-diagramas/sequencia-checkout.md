# Diagrama de Sequência — Checkout até Liquidação

Este diagrama é o que melhor mostra, na prática, a fronteira entre síncrono e assíncrono definida no [ADR-0003](../03-decisoes-arquiteturais/0003-processamento-assincrono-do-split.md): tudo acima da linha pontilhada acontece no orçamento de 3s do comprador; tudo abaixo acontece em até 5 minutos, sem que o comprador espere por isso.

```mermaid
sequenceDiagram
    actor C as Comprador
    participant CK as Checkout/Autorização
    participant ADQ as Adquirente
    participant ES as Event Store
    participant BUS as Barramento (Kafka)
    participant SP as Split/Liquidação
    participant AF as Antifraude
    participant V as Vendedor

    C->>CK: Envia pagamento (carrinho multi-vendedor)
    CK->>ADQ: Solicita autorização
    ADQ-->>CK: Aprovado
    CK->>ES: Grava evento PagamentoAutorizado
    CK-->>C: Confirma compra (< 3s)

    Note over CK,BUS: --- fim do caminho síncrono (P95 < 3s) ---

    CK->>BUS: Publica PagamentoAutorizado
    BUS->>SP: Consome evento
    BUS->>AF: Consome evento

    par Verificação antifraude
        AF->>AF: Calcula score de risco
        AF->>ES: Grava evento AntifraudeAprovado (ou Sinalizado)
    and Cálculo de split
        SP->>SP: Calcula valor por vendedor + comissão
        SP->>ES: Grava evento SplitCalculado
    end

    alt Antifraude aprovado
        SP->>ES: Grava evento AguardandoJanelaDisputa (D+7)
        Note over SP,V: Após D+7 sem disputa aberta
        SP->>ES: Grava evento Liquidada
        SP->>V: Notifica valor disponível
    else Antifraude sinalizou risco
        SP->>ES: Grava evento LiquidacaoBloqueada
        Note over SP: Transação vai para fila de revisão manual (RF-06)
    end
```

**Pontos de decisão visíveis neste diagrama:**
- A resposta ao comprador (`CK-->>C`) acontece **antes** de qualquer split ou antifraude ser calculado — é o que garante o P95 < 3s sem depender da latência de serviços downstream.
- O `Event Store` é escrito por múltiplos serviços de forma independente — cada um contribui com seu próprio evento para a mesma transação, sem um coordenador central bloqueando os outros (Saga coreografada, [ADR-0002](../03-decisoes-arquiteturais/0002-padrao-saga-para-consistencia-transacional.md)).
- O branch `alt` mostra o ponto exato onde RF-06 (antifraude pré-liquidação) intercepta o fluxo antes de qualquer valor ser efetivamente liberado ao vendedor.
