# Data Flow Diagrams

## Member Registration Data Flow

```mermaid
graph TB
    subgraph "User Interface"
        WEB[Web Form]
        MOBILE[Mobile App]
    end
    
    subgraph "API Gateway"
        VALIDATE[Input Validation]
        AUTH_CHECK[Auth Check]
        TRANSFORM[Transform Request]
    end
    
    subgraph "Processing Layer"
        MEMBER_SVC[Member Service]
        PAYMENT_SVC[Payment Service]
        EMAIL_SVC[Email Service]
        AUDIT_SVC[Audit Service]
    end
    
    subgraph "Data Stores"
        LEGACY_DB[(Legacy MySQL)]
        CRM_DB[(CRM Database)]
        CACHE[(Redis Cache)]
        QUEUE[Message Queue]
    end
    
    subgraph "External Services"
        PAYMENT_GW[Payment Gateway]
        EMAIL_PROVIDER[Email Provider]
        CERT_BODY[Certification Body]
    end
    
    %% User flow
    WEB --> VALIDATE
    MOBILE --> VALIDATE
    VALIDATE --> AUTH_CHECK
    AUTH_CHECK --> TRANSFORM
    TRANSFORM --> MEMBER_SVC
    
    %% Member service flow
    MEMBER_SVC --> LEGACY_DB
    MEMBER_SVC --> QUEUE
    MEMBER_SVC --> AUDIT_SVC
    
    %% Async processing
    QUEUE --> CRM_DB
    QUEUE --> EMAIL_SVC
    QUEUE --> PAYMENT_SVC
    
    %% External service calls
    PAYMENT_SVC --> PAYMENT_GW
    EMAIL_SVC --> EMAIL_PROVIDER
    MEMBER_SVC --> CERT_BODY
    
    %% Caching
    MEMBER_SVC --> CACHE
    CACHE --> MEMBER_SVC
    
    %% Audit trail
    AUDIT_SVC --> LEGACY_DB
    
    style WEB fill:#e1f5fe
    style MOBILE fill:#e1f5fe
    style LEGACY_DB fill:#fff3e0
    style CRM_DB fill:#fff3e0
    style PAYMENT_GW fill:#ffebee
    style EMAIL_PROVIDER fill:#ffebee
    style CERT_BODY fill:#ffebee
```

## Payment Processing Data Flow

```mermaid
sequenceDiagram
    participant UI as User Interface
    participant API as API Gateway
    participant PS as Payment Service
    participant LS as Legacy System
    participant PG as Payment Gateway
    participant CRM as CRM System
    participant ES as Email Service
    participant AS as Audit Service
    
    UI->>API: Submit Payment
    API->>API: Validate Request
    API->>PS: Process Payment
    
    PS->>LS: Check Member Status
    LS->>PS: Member Valid
    
    PS->>PG: Charge Card
    PG->>PS: Payment Token
    
    PS->>LS: Update Payment Record
    PS->>AS: Log Transaction
    
    par Async Updates
        PS->>CRM: Update Contact
        PS->>ES: Send Receipt
    end
    
    PS->>API: Payment Success
    API->>UI: Confirmation
    
    Note over PS,CRM: Eventual Consistency
```

## Certification Sync Data Flow

```mermaid
graph LR
    subgraph "Certification Bodies"
        CERT1[Certification Provider 1]
        CERT2[Certification Provider 2]
        CERT3[Certification Provider 3]
    end
    
    subgraph "Integration Layer"
        ADAPTER1[Provider 1 Adapter]
        ADAPTER2[Provider 2 Adapter]
        ADAPTER3[Provider 3 Adapter]
        NORM[Normalizer]
        VALIDATOR[Validator]
    end
    
    subgraph "Processing"
        MATCH[Member Matching]
        MERGE[Data Merge]
        CONFLICT[Conflict Resolution]
    end
    
    subgraph "Storage"
        STAGING[(Staging DB)]
        LEGACY[(Legacy DB)]
        CRM[(CRM)]
        DW[(Data Warehouse)]
    end
    
    %% Provider connections
    CERT1 -->|SOAP| ADAPTER1
    CERT2 -->|REST| ADAPTER2
    CERT3 -->|SFTP| ADAPTER3
    
    %% Normalization
    ADAPTER1 --> NORM
    ADAPTER2 --> NORM
    ADAPTER3 --> NORM
    
    NORM --> VALIDATOR
    VALIDATOR --> STAGING
    
    %% Processing
    STAGING --> MATCH
    LEGACY --> MATCH
    MATCH --> MERGE
    MERGE --> CONFLICT
    
    %% Distribution
    CONFLICT --> LEGACY
    CONFLICT --> CRM
    CONFLICT --> DW
    
    style CERT1 fill:#ffcdd2
    style CERT2 fill:#ffcdd2
    style CERT3 fill:#ffcdd2
```

## Event Stream Data Flow

```mermaid
graph TB
    subgraph "Event Sources"
        WEB_EVENTS[Web Events]
        API_EVENTS[API Events]
        DB_EVENTS[Database CDC]
        SYSTEM_EVENTS[System Events]
    end
    
    subgraph "Event Bus"
        INGEST[Event Ingestion]
        ROUTER[Event Router]
        FILTER[Event Filter]
        ENRICHER[Event Enricher]
    end
    
    subgraph "Event Processing"
        STREAM_PROC[Stream Processor<br/>Apache Flink]
        RULES_ENGINE[Rules Engine]
        ML_PIPELINE[ML Pipeline]
    end
    
    subgraph "Event Consumers"
        NOTIF_SVC[Notification Service]
        ANALYTICS[Analytics Service]
        AUDIT_LOG[Audit Logger]
        SYNC_SVC[Sync Service]
    end
    
    subgraph "Storage"
        EVENT_STORE[(Event Store)]
        METRICS_DB[(Metrics DB)]
        DATA_LAKE[(Data Lake)]
    end
    
    %% Source connections
    WEB_EVENTS --> INGEST
    API_EVENTS --> INGEST
    DB_EVENTS --> INGEST
    SYSTEM_EVENTS --> INGEST
    
    %% Event processing
    INGEST --> ROUTER
    ROUTER --> FILTER
    FILTER --> ENRICHER
    ENRICHER --> STREAM_PROC
    
    %% Processing branches
    STREAM_PROC --> RULES_ENGINE
    STREAM_PROC --> ML_PIPELINE
    
    %% Consumer distribution
    RULES_ENGINE --> NOTIF_SVC
    RULES_ENGINE --> SYNC_SVC
    STREAM_PROC --> ANALYTICS
    STREAM_PROC --> AUDIT_LOG
    
    %% Storage
    STREAM_PROC --> EVENT_STORE
    ANALYTICS --> METRICS_DB
    ML_PIPELINE --> DATA_LAKE
```

## Batch ETL Data Flow

```mermaid
graph TB
    subgraph "Data Sources"
        LEGACY_SRC[(Legacy DB)]
        CRM_SRC[(CRM DB)]
        FILE_SRC[File Exports]
        API_SRC[External APIs]
    end
    
    subgraph "ETL Pipeline"
        EXTRACT[Extract Layer]
        CLEAN[Data Cleansing]
        TRANSFORM[Transformation]
        VALIDATE_ETL[Validation]
        LOAD[Load Layer]
    end
    
    subgraph "Data Quality"
        PROFILE[Data Profiling]
        QUALITY[Quality Rules]
        ANOMALY[Anomaly Detection]
        LINEAGE[Data Lineage]
    end
    
    subgraph "Targets"
        DW_STAGING[(DW Staging)]
        DW_PROD[(Data Warehouse)]
        LAKE[(Data Lake)]
        REPORTS[Reporting Layer]
    end
    
    %% Extract phase
    LEGACY_SRC --> EXTRACT
    CRM_SRC --> EXTRACT
    FILE_SRC --> EXTRACT
    API_SRC --> EXTRACT
    
    %% Transform phase
    EXTRACT --> CLEAN
    CLEAN --> PROFILE
    PROFILE --> QUALITY
    QUALITY --> TRANSFORM
    TRANSFORM --> ANOMALY
    ANOMALY --> VALIDATE_ETL
    VALIDATE_ETL --> LOAD
    
    %% Load phase
    LOAD --> DW_STAGING
    DW_STAGING --> DW_PROD
    LOAD --> LAKE
    DW_PROD --> REPORTS
    
    %% Lineage tracking
    EXTRACT -.-> LINEAGE
    TRANSFORM -.-> LINEAGE
    LOAD -.-> LINEAGE
    
    style LEGACY_SRC fill:#ffe0b2
    style CRM_SRC fill:#ffe0b2
    style DW_PROD fill:#c8e6c9
    style LAKE fill:#c8e6c9
```

## Real-time Member Activity Flow

```mermaid
sequenceDiagram
    participant User
    participant WebApp
    participant API
    participant EventHub
    participant StreamProc as Stream Processor
    participant ML as ML Service
    participant CRM
    participant Notif as Notification Service
    
    User->>WebApp: Perform Action
    WebApp->>API: API Request
    API->>EventHub: Publish Event
    
    par Real-time Processing
        EventHub->>StreamProc: Event Stream
        StreamProc->>ML: Analyze Behavior
        ML->>StreamProc: Insights
        
        alt High-Value Action
            StreamProc->>CRM: Update Score
            StreamProc->>Notif: Trigger Alert
            Notif->>User: Real-time Notification
        else Normal Action
            StreamProc->>CRM: Update Activity
        end
    and Batch Processing
        EventHub->>EventHub: Store for Batch
    end
    
    API->>User: Response
```

## Data Reconciliation Flow

```mermaid
graph TB
    subgraph "Data Sources"
        LEGACY_REC[(Legacy System)]
        CRM_REC[(CRM System)]
        PAYMENT_REC[(Payment System)]
    end
    
    subgraph "Reconciliation Engine"
        EXTRACT_REC[Extract Data]
        COMPARE[Compare Records]
        IDENTIFY[Identify Discrepancies]
        RESOLVE[Resolve Conflicts]
        REPORT_REC[Generate Report]
    end
    
    subgraph "Resolution Actions"
        AUTO_FIX[Auto-Fix<br/>Minor Issues]
        MANUAL_REVIEW[Manual Review<br/>Queue]
        UPDATE_SRC[Update Source]
        NOTIFY_ADMIN[Notify Admin]
    end
    
    subgraph "Outputs"
        RECON_DB[(Reconciliation DB)]
        AUDIT_DB[(Audit Trail)]
        DASHBOARD[Dashboard]
        ALERTS[Alert System]
    end
    
    %% Data extraction
    LEGACY_REC --> EXTRACT_REC
    CRM_REC --> EXTRACT_REC
    PAYMENT_REC --> EXTRACT_REC
    
    %% Reconciliation process
    EXTRACT_REC --> COMPARE
    COMPARE --> IDENTIFY
    IDENTIFY --> RESOLVE
    RESOLVE --> REPORT_REC
    
    %% Resolution routing
    RESOLVE -->|Simple| AUTO_FIX
    RESOLVE -->|Complex| MANUAL_REVIEW
    AUTO_FIX --> UPDATE_SRC
    MANUAL_REVIEW --> UPDATE_SRC
    
    %% Outputs
    UPDATE_SRC --> RECON_DB
    REPORT_REC --> AUDIT_DB
    RECON_DB --> DASHBOARD
    IDENTIFY -->|Critical| NOTIFY_ADMIN
    NOTIFY_ADMIN --> ALERTS
    
    style IDENTIFY fill:#ffeb3b
    style MANUAL_REVIEW fill:#ff9800
    style AUTO_FIX fill:#4caf50
```

## Security Data Flow

```mermaid
graph TB
    subgraph "Security Layers"
        WAF[Web Application Firewall]
        API_SEC[API Security Gateway]
        OAUTH[OAuth2 Server]
        LDAP[LDAP/AD]
    end
    
    subgraph "Security Processing"
        TOKEN_VAL[Token Validation]
        RATE_LIMIT[Rate Limiting]
        THREAT_DET[Threat Detection]
        AUDIT_SEC[Security Audit]
    end
    
    subgraph "Data Protection"
        ENCRYPT[Encryption Service]
        TOKENIZE[Tokenization]
        MASK[Data Masking]
        KEY_MGMT[Key Management]
    end
    
    subgraph "Monitoring"
        SIEM[SIEM System]
        LOG_AGGR[Log Aggregator]
        ALERT_SYS[Alert System]
    end
    
    %% Request flow
    WAF --> API_SEC
    API_SEC --> TOKEN_VAL
    TOKEN_VAL --> OAUTH
    OAUTH --> LDAP
    
    %% Security checks
    API_SEC --> RATE_LIMIT
    API_SEC --> THREAT_DET
    
    %% Data protection
    TOKEN_VAL --> ENCRYPT
    ENCRYPT --> TOKENIZE
    TOKENIZE --> MASK
    ENCRYPT --> KEY_MGMT
    
    %% Monitoring
    WAF --> LOG_AGGR
    API_SEC --> LOG_AGGR
    THREAT_DET --> SIEM
    LOG_AGGR --> SIEM
    SIEM --> ALERT_SYS
    
    %% Audit trail
    TOKEN_VAL --> AUDIT_SEC
    AUDIT_SEC --> LOG_AGGR
    
    style WAF fill:#ff5252
    style THREAT_DET fill:#ff5252
    style SIEM fill:#ff5252
```