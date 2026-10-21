# Platform Architecture Diagram

```mermaid
flowchart LR
    client["Customer / Admin"]

    subgraph platform["Spring Boot REST platform"]
        identity["Identity<br/>:8081"]
        catalog["Catalog<br/>:8082"]
        inventory["Inventory<br/>:8083"]
        cart["Cart<br/>:8084"]
        order["Order<br/>:8085<br/><b>Saga orchestrator</b>"]
        payment["Payment<br/>:8086"]
    end

    subgraph stores["Private H2 databases"]
        identityDb[("Identity H2")]
        catalogDb[("Catalog H2")]
        inventoryDb[("Inventory H2")]
        cartDb[("Cart H2")]
        orderDb[("Order H2")]
        paymentDb[("Payment H2")]
    end

    client -->|"Bearer JWT"| identity
    client -->|"Bearer JWT"| catalog
    client -->|"Bearer JWT"| cart
    client -->|"Bearer JWT"| order

    identity --> identityDb
    catalog --> catalogDb
    inventory --> inventoryDb
    cart --> cartDb
    order --> orderDb
    payment --> paymentDb

    cart -->|"RestClient: product/pricing API"| catalog
    order -->|"RestClient: reserve/release<br/>SERVICE or bearer token"| inventory
    order -->|"RestClient: pay/refund<br/>SERVICE or bearer token"| payment
    order -->|"REST: cart/order inputs"| cart

    request["Every downstream call:<br/>X-Correlation-Id<br/>1 s connect / 2 s read timeout"]
    request -.-> catalog
    request -.-> inventory
    request -.-> payment

    saga["Order saga:<br/>CREATED → STOCK_RESERVED → PAYMENT_PENDING → CONFIRMED<br/>failure paths compensate via inventory release/refund"]
    saga -.-> order

    classDef service fill:#e8f1fb,stroke:#2563a6,color:#111
    classDef db fill:#fff4d6,stroke:#b7791f,color:#111
    classDef note fill:#f1f5f9,stroke:#64748b,color:#111
    class identity,catalog,inventory,cart,order,payment service
    class identityDb,catalogDb,inventoryDb,cartDb,orderDb,paymentDb db
    class request,saga note
```

The databases are private to their owning service. No service reads another
service's database; all cross-service integration uses REST DTOs. Docker,
Kubernetes, Kafka, RabbitMQ, Redis, and external databases are not runtime
dependencies.
