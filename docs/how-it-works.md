# How DBSnapper Works

## Technical Architecture Overview

DBSnapper is built on a **bring-your-own-infrastructure (BYOI)** model that ensures your sensitive data never leaves your control. The architecture consists of three main components working together to provide secure, efficient database snapshot management.

```mermaid
graph TB
    subgraph "Your Infrastructure"
        DB[(Production<br/>Database)]
        Agent[DBSnapper Agent<br/>CLI Tool]
        Storage[Your Cloud Storage<br/>S3/R2/etc]
        
        DB -->|1. Snapshot| Agent
        Agent -->|2. Sanitize| Agent
        Agent -->|3. Upload| Storage
    end
    
    subgraph "DBSnapper Cloud"
        API[DBSnapper API]
        Auth[SSO/Auth Service]
        Meta[Metadata Store]
        
        Agent -.->|Metadata Only| API
        API --> Meta
        API --> Auth
    end
    
    subgraph "Your Team"
        Dev1[Developer 1]
        Dev2[Developer 2]
        CI[CI/CD Pipeline]
        
        Storage -->|4. Download| Dev1
        Storage -->|4. Download| Dev2
        Storage -->|4. Download| CI
    end
    
    Auth -.->|Access Control| Storage
    Meta -.->|Snapshot Info| Dev1
    Meta -.->|Snapshot Info| Dev2
```

### Key Architectural Principles

1. **Data Sovereignty**: Your database snapshots never pass through DBSnapper servers
2. **Zero Trust Security**: All operations are authenticated and authorized
3. **Ephemeral Processing**: No persistent storage of sensitive data during operations
4. **Cloud-Native Design**: Built for modern cloud infrastructure and DevOps workflows

## Detailed Data Flow

### 1. Snapshot Creation Process

```mermaid
sequenceDiagram
    participant U as User
    participant A as DBSnapper Agent
    participant DB as Database
    participant S as Cloud Storage
    participant API as DBSnapper Cloud
    
    U->>A: dbsnapper build target_name
    A->>DB: Connect with credentials
    DB-->>A: Connection established
    
    A->>DB: Create snapshot
    Note over A,DB: Uses native tools<br/>(pg_dump, mysqldump)
    DB-->>A: Snapshot data stream
    
    A->>A: Apply sanitization rules
    Note over A: PII removal<br/>Data masking<br/>Subsetting
    
    A->>S: Upload sanitized snapshot
    S-->>A: Upload confirmation
    
    A->>API: Register snapshot metadata
    Note over API: Only metadata stored:<br/>- Snapshot ID<br/>- Size, timestamp<br/>- No actual data
    API-->>A: Registration complete
    
    A-->>U: Snapshot created successfully
```

### 2. Snapshot Sharing Workflow

```mermaid
sequenceDiagram
    participant U1 as User 1
    participant API as DBSnapper Cloud
    participant Auth as SSO Provider
    participant S as Cloud Storage
    participant U2 as User 2
    
    U1->>API: Share snapshot with team
    API->>Auth: Verify user permissions
    Auth-->>API: User authorized
    
    API->>API: Create access grant
    Note over API: Time-limited<br/>Scoped permissions
    
    U2->>API: List available snapshots
    API->>Auth: Verify user identity
    Auth-->>API: User authenticated
    
    API-->>U2: Available snapshots list
    U2->>API: Request snapshot access
    API->>API: Generate signed URL
    API-->>U2: Temporary download URL
    
    U2->>S: Download snapshot
    S-->>U2: Snapshot data
```

## Security Model

### Multi-Layer Security Architecture

```mermaid
graph LR
    subgraph "Access Control"
        SSO[SSO Integration<br/>OIDC/SAML]
        RBAC[Role-Based<br/>Access Control]
        Groups[SSO Group<br/>Mapping]
    end
    
    subgraph "Data Protection"
        E2E[End-to-End<br/>Encryption]
        AtRest[Encryption<br/>at Rest]
        InTransit[TLS 1.3<br/>in Transit]
    end
    
    subgraph "Compliance"
        Audit[Audit<br/>Logging]
        PII[PII<br/>Detection]
        Rules[Sanitization<br/>Rules Engine]
    end
    
    SSO --> RBAC
    RBAC --> Groups
    Groups --> E2E
    E2E --> AtRest
    AtRest --> InTransit
    InTransit --> Audit
    Audit --> PII
    PII --> Rules
```

### Security Features

1. **Authentication & Authorization**
   - SSO integration (Okta, Azure AD, Google Workspace)
   - Multi-factor authentication support
   - API key management for CI/CD

2. **Data Protection**
   - Client-side encryption before upload
   - Server-side encryption in cloud storage
   - TLS 1.3 for all data transfers

3. **Access Control**
   - Temporary, signed URLs for downloads
   - IP allowlisting
   - Time-based access expiration

4. **Compliance**
   - GDPR-compliant data handling
   - HIPAA-ready sanitization rules
   - SOC 2 Type II practices

## Comparison with Traditional Backup Methods

| Feature | DBSnapper | pg_dump/mysqldump | Custom Scripts | Backup Services |
|---------|-----------|-------------------|----------------|-----------------|
| **Sanitization** | ✅ Built-in, configurable | ❌ Manual post-processing | ⚠️ Varies | ❌ Not included |
| **Subsetting** | ✅ Intelligent, relational | ❌ All or nothing | ⚠️ Complex to implement | ❌ Full backups only |
| **Team Sharing** | ✅ SSO-integrated | ❌ Manual file sharing | ❌ DIY solution | ⚠️ Limited |
| **Automation** | ✅ CI/CD ready | ⚠️ Basic scripting | ⚠️ Custom development | ✅ Some support |
| **Storage Flexibility** | ✅ Any S3-compatible | ❌ Local files | ⚠️ Depends on script | ❌ Vendor lock-in |
| **Compliance Tools** | ✅ GDPR/HIPAA ready | ❌ None | ❌ None | ⚠️ Limited |
| **Performance** | ✅ Optimized streaming | ✅ Native speed | ⚠️ Varies | ✅ Optimized |
| **Version Control** | ✅ Snapshot versioning | ❌ Manual | ❌ Manual | ✅ Some support |

## Bring Your Own Infrastructure Model

### How BYOI Works

```mermaid
graph TB
    subgraph "You Control"
        YDB[(Your Databases)]
        YCS[Your Cloud Storage]
        YCreds[Your Credentials]
        YData[Your Data]
    end
    
    subgraph "DBSnapper Provides"
        Tools[CLI Tools]
        API[Metadata API]
        UI[Web Interface]
        Int[Integrations]
    end
    
    subgraph "Benefits"
        Sov[Data Sovereignty]
        Comp[Compliance]
        Cost[Cost Control]
        Flex[Flexibility]
    end
    
    YDB --> Tools
    YCS --> Tools
    YCreds --> Tools
    YData --> Tools
    
    Tools --> API
    API --> UI
    UI --> Int
    
    Tools --> Sov
    Tools --> Comp
    Tools --> Cost
    Tools --> Flex
```

### BYOI Benefits

1. **Data Sovereignty**
   - Your data never touches DBSnapper servers
   - Complete control over data residency
   - Meet regulatory requirements

2. **Cost Optimization**
   - Use existing cloud storage contracts
   - Leverage reserved capacity
   - No data egress fees to DBSnapper

3. **Security Compliance**
   - Maintain existing security policies
   - Use your encryption keys
   - Integrate with your security tools

4. **Infrastructure Flexibility**
   - Any S3-compatible storage
   - On-premises or cloud databases
   - Hybrid cloud support

## Performance Characteristics

### Snapshot Performance Metrics

```mermaid
graph LR
    subgraph "Small Databases < 10GB"
        S1[Snapshot: 2-5 min]
        S2[Sanitize: 1-3 min]
        S3[Upload: 2-5 min]
        S1 --> S2 --> S3
    end
    
    subgraph "Medium Databases 10-100GB"
        M1[Snapshot: 10-30 min]
        M2[Sanitize: 5-15 min]
        M3[Upload: 10-30 min]
        M1 --> M2 --> M3
    end
    
    subgraph "Large Databases > 100GB"
        L1[Snapshot: 30-120 min]
        L2[Sanitize: 15-60 min]
        L3[Upload: 30-120 min]
        L1 --> L2 --> L3
    end
```

### Performance Optimization Features

1. **Streaming Architecture**
   - Process data without full disk writes
   - Reduced I/O overhead
   - Memory-efficient operations

2. **Parallel Processing**
   - Multi-threaded sanitization
   - Concurrent upload streams
   - Parallel table processing

3. **Incremental Snapshots**
   - Only capture changes
   - Reduce snapshot time
   - Lower storage costs

4. **Compression**
   - Automatic compression
   - Multiple algorithm support
   - Bandwidth optimization

## Compliance and Regulatory Considerations

### Data Protection Compliance

```mermaid
mindmap
  root((Compliance))
    GDPR
      Right to be Forgotten
      Data Minimization
      Purpose Limitation
      Pseudonymization
    HIPAA
      PHI Protection
      Minimum Necessary
      Access Controls
      Audit Trails
    CCPA
      Consumer Rights
      Data Inventory
      Opt-out Support
      Deletion Rights
    SOC2
      Security Controls
      Availability
      Confidentiality
      Privacy
```

### Compliance Features

1. **GDPR Compliance**
   - Automated PII detection and removal
   - Configurable retention policies
   - Data portability support
   - Right to erasure implementation

2. **HIPAA Compliance**
   - PHI sanitization rules
   - Access logging and audit trails
   - Encryption in transit and at rest
   - Business Associate Agreement (BAA) ready

3. **Industry Standards**
   - PCI DSS card data masking
   - Financial data protection
   - Government security requirements
   - Custom compliance rules

## Integration Ecosystem

### Native Integrations

```mermaid
graph TD
    DBS[DBSnapper Core]
    
    subgraph "Development Tools"
        VSC[VSCode Extension]
        GHA[GitHub Actions]
        GL[GitLab CI]
        JNK[Jenkins]
    end
    
    subgraph "Infrastructure"
        TF[Terraform Provider]
        K8S[Kubernetes Operator]
        HELM[Helm Charts]
        ANS[Ansible Playbooks]
    end
    
    subgraph "Databases"
        PG[PostgreSQL]
        MY[MySQL]
        MS[SQL Server Beta]
        MG[MongoDB Soon]
    end
    
    subgraph "Storage"
        S3[Amazon S3]
        R2[Cloudflare R2]
        GCS[Google Cloud Storage]
        AZ[Azure Blob]
    end
    
    DBS --> Development Tools
    DBS --> Infrastructure
    DBS --> Databases
    DBS --> Storage
```

## Getting Started

Ready to implement DBSnapper in your workflow? Here's your path forward:

1. **[Install DBSnapper Agent](installation.md)** - Get the CLI tool running
2. **[Configure Your First Target](quick-start.md)** - Connect to your database
3. **[Create a Snapshot](snapshot/introduction.md)** - Build your first snapshot
4. **[Set Up Sanitization](sanitize/introduction.md)** - Remove sensitive data
5. **[Share with Your Team](share/introduction.md)** - Collaborate securely

For enterprise deployments, explore [DBSnapper Cloud](dbsnapper-cloud/introduction.md) for advanced team features and SSO integration.