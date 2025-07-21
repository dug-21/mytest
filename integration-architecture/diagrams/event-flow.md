# Event Flow Diagrams

## Real-Time Member Update Event Flow

```mermaid
sequenceDiagram
    participant User
    participant WebApp
    participant API
    participant EventBus
    participant CRM
    participant Email
    participant Audit
    participant Analytics
    
    User->>WebApp: Update Profile
    WebApp->>API: PUT /members/{id}
    API->>API: Validate Changes
    API->>EventBus: Publish member.profile.updated
    
    par Synchronous Response
        API->>WebApp: 200 OK
        WebApp->>User: Update Confirmed
    and Asynchronous Processing
        EventBus->>CRM: Update Contact
        EventBus->>Email: Send Confirmation
        EventBus->>Audit: Log Change
        EventBus->>Analytics: Track Activity
    end
    
    CRM->>EventBus: Publish sync.completed
    Email->>EventBus: Publish email.sent
    
    Note over EventBus: All events stored for replay
```

## Payment Processing Event Flow

```mermaid
graph TB
    subgraph "Payment Initiation"
        USER[User] --> CHECKOUT[Checkout Page]
        CHECKOUT --> PAYMENT_API[Payment API]
    end
    
    subgraph "Event Generation"
        PAYMENT_API --> INIT_EVENT[payment.initiated]
        INIT_EVENT --> EVENT_BUS[Event Bus]
    end
    
    subgraph "Payment Processing"
        EVENT_BUS --> PAYMENT_SVC[Payment Service]
        PAYMENT_SVC --> GATEWAY[Payment Gateway]
        GATEWAY --> AUTH_EVENT[payment.authorized]
        AUTH_EVENT --> EVENT_BUS
    end
    
    subgraph "Downstream Processing"
        EVENT_BUS --> INVENTORY[Inventory Service]
        EVENT_BUS --> MEMBERSHIP[Membership Service]
        EVENT_BUS --> NOTIFICATION[Notification Service]
        EVENT_BUS --> ACCOUNTING[Accounting Service]
    end
    
    subgraph "Completion Events"
        INVENTORY --> INV_EVENT[inventory.reserved]
        MEMBERSHIP --> MEM_EVENT[membership.activated]
        NOTIFICATION --> NOTIF_EVENT[notification.sent]
        ACCOUNTING --> ACC_EVENT[transaction.recorded]
    end
    
    INV_EVENT --> EVENT_BUS
    MEM_EVENT --> EVENT_BUS
    NOTIF_EVENT --> EVENT_BUS
    ACC_EVENT --> EVENT_BUS
    
    style USER fill:#e1f5fe
    style EVENT_BUS fill:#fff3e0
    style GATEWAY fill:#ffebee
```

## Certification Lifecycle Event Flow

```mermaid
stateDiagram-v2
    [*] --> Applied: certification.applied
    Applied --> UnderReview: certification.review.started
    UnderReview --> Approved: certification.approved
    UnderReview --> Rejected: certification.rejected
    Approved --> Issued: certification.issued
    Issued --> Active: certification.activated
    Active --> Expiring: certification.expiring_soon
    Expiring --> Renewed: certification.renewed
    Expiring --> Expired: certification.expired
    Active --> Suspended: certification.suspended
    Suspended --> Active: certification.reactivated
    Suspended --> Revoked: certification.revoked
    Renewed --> Active: certification.activated
    
    Rejected --> [*]
    Expired --> [*]
    Revoked --> [*]
    
    note right of Applied
        Event: certification.applied
        Data: applicantId, certType, documents
    end note
    
    note right of Issued
        Event: certification.issued
        Data: certNumber, issueDate, expiryDate
    end note
    
    note right of Renewed
        Event: certification.renewed
        Data: newExpiryDate, renewalFee
    end note
```

## Event Aggregation Flow

```mermaid
graph LR
    subgraph "Raw Events"
        E1[Login Event]
        E2[Profile View]
        E3[Course Start]
        E4[Course Complete]
        E5[Payment Made]
    end
    
    subgraph "Stream Processing"
        WINDOW[5-min Window]
        AGGREGATE[Aggregator]
        PATTERN[Pattern Detector]
    end
    
    subgraph "Aggregated Events"
        ACTIVITY[member.activity.summary]
        ENGAGEMENT[member.engagement.score]
        RISK[member.risk.assessment]
    end
    
    subgraph "Actions"
        REWARD[Trigger Reward]
        ALERT[Security Alert]
        RECOMMEND[Recommendations]
    end
    
    E1 --> WINDOW
    E2 --> WINDOW
    E3 --> WINDOW
    E4 --> WINDOW
    E5 --> WINDOW
    
    WINDOW --> AGGREGATE
    AGGREGATE --> PATTERN
    
    PATTERN --> ACTIVITY
    PATTERN --> ENGAGEMENT
    PATTERN --> RISK
    
    ACTIVITY --> RECOMMEND
    ENGAGEMENT --> REWARD
    RISK --> ALERT
```

## Saga Pattern for Distributed Transactions

```mermaid
sequenceDiagram
    participant Client
    participant OrderSaga
    participant Payment
    participant Inventory
    participant Shipping
    participant EventBus
    
    Client->>OrderSaga: Create Order
    OrderSaga->>EventBus: order.created
    
    EventBus->>Payment: Process Payment
    Payment->>Payment: Charge Card
    alt Payment Success
        Payment->>EventBus: payment.completed
        EventBus->>OrderSaga: Payment OK
        
        OrderSaga->>EventBus: inventory.reserve
        EventBus->>Inventory: Reserve Items
        Inventory->>Inventory: Check Stock
        
        alt Stock Available
            Inventory->>EventBus: inventory.reserved
            EventBus->>OrderSaga: Inventory OK
            
            OrderSaga->>EventBus: shipping.create
            EventBus->>Shipping: Create Shipment
            Shipping->>EventBus: shipment.created
            EventBus->>OrderSaga: Shipping OK
            
            OrderSaga->>EventBus: order.completed
            OrderSaga->>Client: Order Successful
        else Stock Unavailable
            Inventory->>EventBus: inventory.failed
            EventBus->>OrderSaga: Inventory Failed
            
            Note over OrderSaga: Start Compensation
            OrderSaga->>EventBus: payment.refund
            EventBus->>Payment: Refund Payment
            Payment->>EventBus: payment.refunded
            
            OrderSaga->>EventBus: order.cancelled
            OrderSaga->>Client: Order Failed
        end
    else Payment Failed
        Payment->>EventBus: payment.failed
        EventBus->>OrderSaga: Payment Failed
        OrderSaga->>EventBus: order.failed
        OrderSaga->>Client: Order Failed
    end
```

## Event Sourcing Data Flow

```mermaid
graph TB
    subgraph "Commands"
        CMD1[Create Member]
        CMD2[Update Profile]
        CMD3[Renew Membership]
    end
    
    subgraph "Command Handler"
        VALIDATE[Validate Command]
        LOAD[Load Aggregate]
        APPLY[Apply Business Logic]
        EMIT[Emit Events]
    end
    
    subgraph "Event Store"
        APPEND[Append Events]
        EVENTS[(Event Stream)]
        SNAPSHOT[(Snapshots)]
    end
    
    subgraph "Projections"
        READ_MODEL[Read Model]
        SEARCH_INDEX[Search Index]
        ANALYTICS_DB[Analytics DB]
        CACHE[Cache Layer]
    end
    
    subgraph "Query Side"
        API_QUERY[Query API]
        REPORTS[Reports]
        DASHBOARD[Dashboard]
    end
    
    CMD1 --> VALIDATE
    CMD2 --> VALIDATE
    CMD3 --> VALIDATE
    
    VALIDATE --> LOAD
    LOAD --> EVENTS
    LOAD --> SNAPSHOT
    LOAD --> APPLY
    APPLY --> EMIT
    EMIT --> APPEND
    APPEND --> EVENTS
    
    EVENTS --> READ_MODEL
    EVENTS --> SEARCH_INDEX
    EVENTS --> ANALYTICS_DB
    READ_MODEL --> CACHE
    
    API_QUERY --> CACHE
    API_QUERY --> READ_MODEL
    REPORTS --> ANALYTICS_DB
    DASHBOARD --> SEARCH_INDEX
    
    style EVENTS fill:#fff3e0
    style READ_MODEL fill:#e8f5e9
    style CACHE fill:#e1f5fe
```

## Complex Event Processing (CEP) Flow

```mermaid
graph TB
    subgraph "Event Sources"
        LOGIN[Login Events]
        ACTIVITY[Activity Events]
        PAYMENT[Payment Events]
        SUPPORT[Support Events]
    end
    
    subgraph "CEP Engine"
        FILTER[Event Filter]
        WINDOW[Time Window]
        CORRELATE[Correlation]
        PATTERN_MATCH[Pattern Matching]
        RULES[Business Rules]
    end
    
    subgraph "Patterns Detected"
        CHURN_RISK[Churn Risk Pattern]
        VIP_BEHAVIOR[VIP Behavior Pattern]
        FRAUD_PATTERN[Fraud Pattern]
        ENGAGEMENT[High Engagement Pattern]
    end
    
    subgraph "Actions"
        RETENTION[Retention Campaign]
        VIP_UPGRADE[VIP Upgrade Offer]
        SECURITY[Security Alert]
        REWARDS[Reward Points]
    end
    
    LOGIN --> FILTER
    ACTIVITY --> FILTER
    PAYMENT --> FILTER
    SUPPORT --> FILTER
    
    FILTER --> WINDOW
    WINDOW --> CORRELATE
    CORRELATE --> PATTERN_MATCH
    PATTERN_MATCH --> RULES
    
    RULES --> CHURN_RISK
    RULES --> VIP_BEHAVIOR
    RULES --> FRAUD_PATTERN
    RULES --> ENGAGEMENT
    
    CHURN_RISK --> RETENTION
    VIP_BEHAVIOR --> VIP_UPGRADE
    FRAUD_PATTERN --> SECURITY
    ENGAGEMENT --> REWARDS
    
    style CEP Engine fill:#fff3e0
    style PATTERN_MATCH fill:#ffeb3b
    style FRAUD_PATTERN fill:#ff5252
```

## Event-Driven Microservices Communication

```mermaid
graph TB
    subgraph "Member Service"
        MS_API[Member API]
        MS_DB[(Member DB)]
        MS_EVENTS[Event Publisher]
    end
    
    subgraph "Event Bus Infrastructure"
        TOPICS[Kafka Topics]
        SCHEMA[Schema Registry]
        MONITOR[Event Monitor]
    end
    
    subgraph "Consuming Services"
        subgraph "CRM Service"
            CRM_CONSUMER[Event Consumer]
            CRM_HANDLER[Event Handler]
            CRM_DB[(CRM DB)]
        end
        
        subgraph "Email Service"
            EMAIL_CONSUMER[Event Consumer]
            EMAIL_HANDLER[Event Handler]
            EMAIL_QUEUE[Email Queue]
        end
        
        subgraph "Analytics Service"
            ANALYTICS_CONSUMER[Event Consumer]
            ANALYTICS_PROCESSOR[Stream Processor]
            ANALYTICS_STORE[(Analytics Store)]
        end
    end
    
    MS_API --> MS_DB
    MS_API --> MS_EVENTS
    MS_EVENTS --> TOPICS
    MS_EVENTS --> SCHEMA
    
    TOPICS --> CRM_CONSUMER
    TOPICS --> EMAIL_CONSUMER
    TOPICS --> ANALYTICS_CONSUMER
    
    CRM_CONSUMER --> CRM_HANDLER
    CRM_HANDLER --> CRM_DB
    
    EMAIL_CONSUMER --> EMAIL_HANDLER
    EMAIL_HANDLER --> EMAIL_QUEUE
    
    ANALYTICS_CONSUMER --> ANALYTICS_PROCESSOR
    ANALYTICS_PROCESSOR --> ANALYTICS_STORE
    
    MONITOR --> TOPICS
    
    style TOPICS fill:#e1f5fe
    style SCHEMA fill:#fff3e0
```

## Event Replay and Recovery Flow

```mermaid
sequenceDiagram
    participant Admin
    participant ReplayService
    participant EventStore
    participant MessageBus
    participant Service1
    participant Service2
    participant Monitor
    
    Admin->>ReplayService: Initiate Replay
    Note over Admin,ReplayService: Start: 2024-01-15 00:00<br/>End: 2024-01-15 23:59
    
    ReplayService->>EventStore: Query Events in Range
    EventStore->>ReplayService: Return Event Batch
    
    ReplayService->>MessageBus: Create Replay Topic
    
    loop For Each Event Batch
        ReplayService->>ReplayService: Add Replay Headers
        ReplayService->>MessageBus: Publish to Replay Topic
        
        par Service Processing
            MessageBus->>Service1: Deliver Event
            Service1->>Service1: Process with Replay Flag
            Service1->>Monitor: Report Progress
        and
            MessageBus->>Service2: Deliver Event
            Service2->>Service2: Process with Replay Flag
            Service2->>Monitor: Report Progress
        end
    end
    
    Monitor->>ReplayService: All Events Processed
    ReplayService->>MessageBus: Delete Replay Topic
    ReplayService->>Admin: Replay Complete
    
    Note over Admin,Monitor: Replay Statistics:<br/>Events: 45,231<br/>Duration: 2h 15m<br/>Errors: 0
```