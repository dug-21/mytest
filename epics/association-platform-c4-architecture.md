# Association Management Platform - C4 Architecture

## Executive Summary

This document presents the C4 architecture for a legacy home-grown web-based community platform integrated with a Salesforce-style CRM. The platform serves as a SaaS backend provider to professional associations (nursing associations, W3C, tech associations, etc.), supporting 50+ associations with over 500,000 total members.

## Table of Contents

1. [Business Context](#business-context)
2. [System Context Diagram (C1)](#system-context-diagram-c1)
3. [Container Diagram (C2)](#container-diagram-c2)
4. [Component Diagrams (C3)](#component-diagrams-c3)
5. [Code Structure (C4)](#code-structure-c4)
6. [Business Process Flows](#business-process-flows)
7. [Integration Patterns](#integration-patterns)
8. [Security Architecture](#security-architecture)
9. [Data Architecture](#data-architecture)

## Business Context

### Core Business Model
- **SaaS Provider**: Multi-tenant platform serving professional associations
- **Revenue Streams**: Subscription fees, transaction fees, premium features
- **Target Market**: Professional associations requiring member management, event coordination, certification tracking, and community engagement

### Key Features
- Member Management & Directory
- Event Registration & Management
- Certification & Continuing Education Tracking
- Content Management & Digital Library
- Community Forums & Networking
- Email Marketing & Communications
- Financial Management & Dues Collection
- Reporting & Analytics

## System Context Diagram (C1)

The System Context diagram shows how the Association Management Platform fits into the broader ecosystem and interacts with external users and systems.

```mermaid
graph TB
    subgraph "Professional Association Ecosystem"
        AssocAdmin[Association Administrators]
        Member[Association Members]
        Guest[Non-Members/Guests]
        Partner[Partners/Sponsors]
    end
    
    subgraph "Association Management Platform"
        AMP[Association Management Platform<br/>SaaS multi-tenant platform for<br/>professional association management]
    end
    
    subgraph "External Systems"
        Payment[Payment Processors<br/>Stripe, PayPal, Authorize.net]
        Email[Email Service Providers<br/>SendGrid, AWS SES]
        SMS[SMS Gateway<br/>Twilio]
        CDN[Content Delivery Network<br/>CloudFlare, AWS CloudFront]
        Analytics[Analytics Services<br/>Google Analytics, Mixpanel]
        Storage[Cloud Storage<br/>AWS S3, Azure Blob]
        LMS[External LMS<br/>Moodle, Canvas]
        ERP[Association ERP Systems<br/>NetSuite, SAP]
        Social[Social Media Platforms<br/>LinkedIn, Twitter, Facebook]
        Webinar[Webinar Platforms<br/>Zoom, GoToWebinar]
    end
    
    AssocAdmin -->|Manages associations,<br/>members, events| AMP
    Member -->|Accesses community,<br/>registers for events,<br/>manages profile| AMP
    Guest -->|Browses public content,<br/>registers as member| AMP
    Partner -->|Manages sponsorships,<br/>accesses metrics| AMP
    
    AMP -->|Processes payments| Payment
    AMP -->|Sends transactional<br/>and marketing emails| Email
    AMP -->|Sends SMS notifications| SMS
    AMP -->|Delivers static content| CDN
    AMP -->|Tracks user behavior| Analytics
    AMP -->|Stores files and media| Storage
    AMP -->|Syncs learning records| LMS
    AMP -->|Syncs financial data| ERP
    AMP -->|Posts updates,<br/>enables SSO| Social
    AMP -->|Hosts virtual events| Webinar
    
    style AMP fill:#4A90E2,stroke:#2E5C8A,stroke-width:4px,color:#fff
```

## Container Diagram (C2)

The Container diagram zooms into the Association Management Platform to show the high-level technical building blocks and their interactions.

```mermaid
graph TB
    subgraph "User Interfaces"
        WebApp[Member Web Application<br/>React SPA]
        AdminPortal[Admin Portal<br/>React + Redux]
        MobileApp[Mobile Applications<br/>React Native]
        PublicSite[Public Website<br/>Next.js SSR]
    end
    
    subgraph "API Gateway Layer"
        Gateway[API Gateway<br/>Kong/AWS API Gateway<br/>Rate limiting, Auth, Routing]
    end
    
    subgraph "Core Services"
        AuthService[Authentication Service<br/>Node.js/Express<br/>JWT, OAuth2, SAML]
        MemberService[Member Service<br/>Java Spring Boot<br/>Member management]
        EventService[Event Service<br/>Node.js/Express<br/>Event registration]
        ContentService[Content Service<br/>Python Django<br/>CMS, Digital library]
        CommunityService[Community Service<br/>Node.js/Express<br/>Forums, networking]
        CRMService[CRM Service<br/>Java Spring Boot<br/>Salesforce-style CRM]
        BillingService[Billing Service<br/>Node.js/Express<br/>Payments, invoicing]
        NotificationService[Notification Service<br/>Node.js/Express<br/>Email, SMS, Push]
        ReportingService[Reporting Service<br/>Python Flask<br/>Analytics, dashboards]
        IntegrationService[Integration Service<br/>Node.js/Express<br/>External system connectors]
    end
    
    subgraph "Data Layer"
        PostgresMain[(PostgreSQL<br/>Primary Database<br/>Members, Events, Content)]
        MongoDB[(MongoDB<br/>Document Store<br/>Forums, Activity Logs)]
        Redis[(Redis<br/>Cache & Sessions<br/>Performance optimization)]
        Elasticsearch[(Elasticsearch<br/>Search Engine<br/>Full-text search)]
        DataWarehouse[(Data Warehouse<br/>Snowflake/Redshift<br/>Analytics, Reporting)]
    end
    
    subgraph "Infrastructure Services"
        Queue[Message Queue<br/>RabbitMQ/AWS SQS<br/>Async processing]
        FileStorage[File Storage<br/>AWS S3<br/>Documents, Media]
        CDN[CDN<br/>CloudFlare<br/>Static assets]
        Monitoring[Monitoring<br/>Datadog/New Relic<br/>APM, Logs, Metrics]
    end
    
    WebApp --> Gateway
    AdminPortal --> Gateway
    MobileApp --> Gateway
    PublicSite --> Gateway
    
    Gateway --> AuthService
    Gateway --> MemberService
    Gateway --> EventService
    Gateway --> ContentService
    Gateway --> CommunityService
    Gateway --> CRMService
    Gateway --> BillingService
    Gateway --> NotificationService
    Gateway --> ReportingService
    Gateway --> IntegrationService
    
    AuthService --> Redis
    AuthService --> PostgresMain
    
    MemberService --> PostgresMain
    MemberService --> Redis
    MemberService --> Queue
    
    EventService --> PostgresMain
    EventService --> Queue
    EventService --> BillingService
    
    ContentService --> PostgresMain
    ContentService --> FileStorage
    ContentService --> CDN
    ContentService --> Elasticsearch
    
    CommunityService --> MongoDB
    CommunityService --> PostgresMain
    CommunityService --> Elasticsearch
    
    CRMService --> PostgresMain
    CRMService --> DataWarehouse
    
    BillingService --> PostgresMain
    BillingService --> Queue
    
    NotificationService --> Queue
    NotificationService --> MongoDB
    
    ReportingService --> DataWarehouse
    ReportingService --> PostgresMain
    
    IntegrationService --> Queue
    IntegrationService --> Redis
    
    Queue --> NotificationService
    Queue --> IntegrationService
    
    style Gateway fill:#FF9800,stroke:#E65100,stroke-width:3px
    style PostgresMain fill:#4CAF50,stroke:#2E7D32,stroke-width:3px
```

## Component Diagrams (C3)

### Member Service Component Diagram

The Member Service is a critical container managing all member-related operations. Here's its internal component structure:

```mermaid
graph TB
    subgraph "Member Service Components"
        subgraph "API Layer"
            MemberAPI[Member REST API<br/>Express Router<br/>RESTful endpoints]
            GraphQLAPI[GraphQL API<br/>Apollo Server<br/>Flexible queries]
        end
        
        subgraph "Business Logic Layer"
            MemberController[Member Controller<br/>Request handling<br/>Validation]
            ProfileManager[Profile Manager<br/>Profile CRUD<br/>Avatar management]
            MembershipManager[Membership Manager<br/>Membership types<br/>Renewal logic]
            DirectoryManager[Directory Manager<br/>Search & filters<br/>Privacy controls]
            ImportExportManager[Import/Export Manager<br/>Bulk operations<br/>Data migration]
            MemberValidator[Member Validator<br/>Data validation<br/>Business rules]
        end
        
        subgraph "Integration Layer"
            AuthIntegration[Auth Integration<br/>Token validation<br/>Permission checks]
            CRMIntegration[CRM Integration<br/>Sync member data<br/>Update activities]
            BillingIntegration[Billing Integration<br/>Dues management<br/>Payment status]
            NotificationIntegration[Notification Integration<br/>Welcome emails<br/>Renewal reminders]
        end
        
        subgraph "Data Access Layer"
            MemberRepository[Member Repository<br/>Data persistence<br/>Query optimization]
            CacheManager[Cache Manager<br/>Redis caching<br/>Performance optimization]
            AuditLogger[Audit Logger<br/>Change tracking<br/>Compliance logging]
        end
    end
    
    MemberAPI --> MemberController
    GraphQLAPI --> MemberController
    
    MemberController --> ProfileManager
    MemberController --> MembershipManager
    MemberController --> DirectoryManager
    MemberController --> ImportExportManager
    MemberController --> MemberValidator
    
    ProfileManager --> MemberRepository
    ProfileManager --> CacheManager
    ProfileManager --> AuditLogger
    
    MembershipManager --> MemberRepository
    MembershipManager --> BillingIntegration
    MembershipManager --> NotificationIntegration
    
    DirectoryManager --> MemberRepository
    DirectoryManager --> CacheManager
    DirectoryManager --> AuthIntegration
    
    ImportExportManager --> MemberRepository
    ImportExportManager --> MemberValidator
    ImportExportManager --> CRMIntegration
    
    MemberValidator --> AuthIntegration
    
    style MemberAPI fill:#FFC107,stroke:#F57C00,stroke-width:2px
    style MemberController fill:#2196F3,stroke:#0D47A1,stroke-width:2px
    style MemberRepository fill:#4CAF50,stroke:#1B5E20,stroke-width:2px
```

### Event Service Component Diagram

The Event Service manages all event-related functionality including registration, scheduling, and virtual event integration.

```mermaid
graph TB
    subgraph "Event Service Components"
        subgraph "API Layer"
            EventAPI[Event REST API<br/>Express Router<br/>Event endpoints]
            WebhookHandler[Webhook Handler<br/>External triggers<br/>Payment callbacks]
        end
        
        subgraph "Business Logic Layer"
            EventController[Event Controller<br/>Request orchestration<br/>Response formatting]
            EventManager[Event Manager<br/>Event CRUD<br/>Scheduling logic]
            RegistrationManager[Registration Manager<br/>Attendee registration<br/>Waitlist management]
            SessionManager[Session Manager<br/>Tracks & sessions<br/>Speaker management]
            VenueManager[Venue Manager<br/>Location management<br/>Room allocation]
            VirtualEventManager[Virtual Event Manager<br/>Webinar integration<br/>Stream management]
            TicketingManager[Ticketing Manager<br/>Ticket types<br/>Pricing rules]
        end
        
        subgraph "Integration Layer"
            PaymentIntegration[Payment Integration<br/>Process payments<br/>Refund handling]
            CalendarIntegration[Calendar Integration<br/>iCal export<br/>Google Calendar sync]
            WebinarIntegration[Webinar Integration<br/>Zoom/Teams API<br/>Session creation]
            EmailIntegration[Email Integration<br/>Confirmations<br/>Reminders]
            BadgeIntegration[Badge Integration<br/>QR codes<br/>Check-in system]
        end
        
        subgraph "Data Access Layer"
            EventRepository[Event Repository<br/>Event persistence<br/>Complex queries]
            RegistrationRepository[Registration Repository<br/>Attendee data<br/>Transaction handling]
            EventCache[Event Cache<br/>Popular events<br/>Session data]
            ReportGenerator[Report Generator<br/>Attendance reports<br/>Revenue analytics]
        end
    end
    
    EventAPI --> EventController
    WebhookHandler --> EventController
    
    EventController --> EventManager
    EventController --> RegistrationManager
    EventController --> SessionManager
    EventController --> VenueManager
    EventController --> VirtualEventManager
    EventController --> TicketingManager
    
    EventManager --> EventRepository
    EventManager --> EventCache
    EventManager --> CalendarIntegration
    
    RegistrationManager --> RegistrationRepository
    RegistrationManager --> PaymentIntegration
    RegistrationManager --> EmailIntegration
    RegistrationManager --> BadgeIntegration
    
    SessionManager --> EventRepository
    SessionManager --> WebinarIntegration
    
    VirtualEventManager --> WebinarIntegration
    VirtualEventManager --> EventRepository
    
    TicketingManager --> EventRepository
    TicketingManager --> PaymentIntegration
    
    EventRepository --> ReportGenerator
    RegistrationRepository --> ReportGenerator
    
    style EventController fill:#9C27B0,stroke:#4A148C,stroke-width:2px
    style RegistrationManager fill:#FF5722,stroke:#BF360C,stroke-width:2px
```

### CRM Service Component Diagram

The CRM Service provides Salesforce-style functionality for managing contacts, opportunities, and organizational relationships.

```mermaid
graph TB
    subgraph "CRM Service Components"
        subgraph "API Layer"
            CRMAPI[CRM REST API<br/>Spring REST<br/>CRUD operations]
            BulkAPI[Bulk API<br/>Batch operations<br/>Import/Export]
            RealtimeAPI[Realtime API<br/>WebSocket<br/>Live updates]
        end
        
        subgraph "Business Logic Layer"
            ContactManager[Contact Manager<br/>Contact lifecycle<br/>Deduplication]
            OrganizationManager[Organization Manager<br/>Company profiles<br/>Hierarchies]
            OpportunityManager[Opportunity Manager<br/>Sales pipeline<br/>Stage management]
            ActivityManager[Activity Manager<br/>Tasks & events<br/>Timeline tracking]
            CampaignManager[Campaign Manager<br/>Marketing campaigns<br/>ROI tracking]
            LeadManager[Lead Manager<br/>Lead scoring<br/>Assignment rules]
            WorkflowEngine[Workflow Engine<br/>Automation rules<br/>Trigger management]
        end
        
        subgraph "Analytics Layer"
            PipelineAnalytics[Pipeline Analytics<br/>Funnel analysis<br/>Conversion rates]
            EngagementScoring[Engagement Scoring<br/>Activity scoring<br/>Predictive analytics]
            RevenueForecasting[Revenue Forecasting<br/>Predictive models<br/>Trend analysis]
        end
        
        subgraph "Data Layer"
            CRMRepository[CRM Repository<br/>JPA/Hibernate<br/>Complex relationships]
            SearchIndex[Search Index<br/>Elasticsearch<br/>Full-text search]
            ChangeDataCapture[Change Data Capture<br/>Event sourcing<br/>Audit trail]
            DataSyncManager[Data Sync Manager<br/>External sync<br/>Conflict resolution]
        end
    end
    
    CRMAPI --> ContactManager
    CRMAPI --> OrganizationManager
    CRMAPI --> OpportunityManager
    CRMAPI --> ActivityManager
    BulkAPI --> DataSyncManager
    RealtimeAPI --> ChangeDataCapture
    
    ContactManager --> CRMRepository
    ContactManager --> SearchIndex
    ContactManager --> WorkflowEngine
    
    OrganizationManager --> CRMRepository
    OrganizationManager --> SearchIndex
    
    OpportunityManager --> CRMRepository
    OpportunityManager --> PipelineAnalytics
    OpportunityManager --> WorkflowEngine
    
    ActivityManager --> CRMRepository
    ActivityManager --> EngagementScoring
    ActivityManager --> ChangeDataCapture
    
    CampaignManager --> CRMRepository
    CampaignManager --> RevenueForecasting
    
    LeadManager --> CRMRepository
    LeadManager --> WorkflowEngine
    LeadManager --> EngagementScoring
    
    WorkflowEngine --> ChangeDataCapture
    WorkflowEngine --> DataSyncManager
    
    PipelineAnalytics --> CRMRepository
    EngagementScoring --> CRMRepository
    RevenueForecasting --> CRMRepository
    
    style CRMAPI fill:#3F51B5,stroke:#1A237E,stroke-width:2px
    style WorkflowEngine fill:#009688,stroke:#00695C,stroke-width:2px
```

## Code Structure (C4)

### Example: Member Profile Manager Component

Here's a detailed code structure for the Profile Manager component within the Member Service:

```mermaid
classDiagram
    class ProfileManager {
        -memberRepository: MemberRepository
        -cacheManager: CacheManager
        -auditLogger: AuditLogger
        -fileStorage: FileStorageService
        +getProfile(memberId: string): Promise~MemberProfile~
        +updateProfile(memberId: string, data: ProfileUpdateDto): Promise~MemberProfile~
        +uploadAvatar(memberId: string, file: File): Promise~string~
        +deleteAvatar(memberId: string): Promise~void~
        +updatePrivacySettings(memberId: string, settings: PrivacySettings): Promise~void~
        +generateProfileCompleteness(profile: MemberProfile): number
        -validateProfileData(data: ProfileUpdateDto): ValidationResult
        -processAvatarImage(file: File): Promise~ProcessedImage~
        -invalidateCache(memberId: string): Promise~void~
    }
    
    class MemberProfile {
        +id: string
        +firstName: string
        +lastName: string
        +email: string
        +phone: string
        +avatarUrl: string
        +bio: string
        +credentials: Credential[]
        +socialLinks: SocialLink[]
        +privacy: PrivacySettings
        +completeness: number
        +createdAt: Date
        +updatedAt: Date
    }
    
    class ProfileUpdateDto {
        +firstName?: string
        +lastName?: string
        +phone?: string
        +bio?: string
        +credentials?: Credential[]
        +socialLinks?: SocialLink[]
    }
    
    class PrivacySettings {
        +showEmail: boolean
        +showPhone: boolean
        +showInDirectory: boolean
        +allowMessages: boolean
        +publicProfile: boolean
    }
    
    class Credential {
        +type: string
        +title: string
        +issuer: string
        +dateIssued: Date
        +expiryDate?: Date
        +verificationUrl?: string
    }
    
    class SocialLink {
        +platform: string
        +url: string
        +visible: boolean
    }
    
    class MemberRepository {
        +findById(id: string): Promise~Member~
        +update(id: string, data: Partial~Member~): Promise~Member~
        +exists(id: string): Promise~boolean~
    }
    
    class CacheManager {
        +get(key: string): Promise~any~
        +set(key: string, value: any, ttl?: number): Promise~void~
        +invalidate(key: string): Promise~void~
    }
    
    class AuditLogger {
        +logProfileUpdate(memberId: string, changes: any): Promise~void~
        +logAvatarChange(memberId: string, action: string): Promise~void~
    }
    
    ProfileManager --> MemberProfile
    ProfileManager --> ProfileUpdateDto
    ProfileManager --> PrivacySettings
    ProfileManager --> MemberRepository
    ProfileManager --> CacheManager
    ProfileManager --> AuditLogger
    MemberProfile --> Credential
    MemberProfile --> SocialLink
    MemberProfile --> PrivacySettings
    ProfileUpdateDto --> Credential
    ProfileUpdateDto --> SocialLink
```

### Example Implementation (TypeScript/Node.js)

```typescript
// ProfileManager.ts
import { injectable, inject } from 'inversify';
import { MemberRepository } from '../repositories/MemberRepository';
import { CacheManager } from '../services/CacheManager';
import { AuditLogger } from '../services/AuditLogger';
import { FileStorageService } from '../services/FileStorageService';
import { MemberProfile, ProfileUpdateDto, PrivacySettings } from '../models';
import { ValidationResult, ProfileValidator } from '../validators/ProfileValidator';

@injectable()
export class ProfileManager {
    constructor(
        @inject('MemberRepository') private memberRepository: MemberRepository,
        @inject('CacheManager') private cacheManager: CacheManager,
        @inject('AuditLogger') private auditLogger: AuditLogger,
        @inject('FileStorageService') private fileStorage: FileStorageService
    ) {}

    async getProfile(memberId: string): Promise<MemberProfile> {
        // Check cache first
        const cacheKey = `profile:${memberId}`;
        const cached = await this.cacheManager.get(cacheKey);
        if (cached) {
            return cached;
        }

        // Fetch from database
        const member = await this.memberRepository.findById(memberId);
        if (!member) {
            throw new Error(`Member ${memberId} not found`);
        }

        const profile = this.mapToProfile(member);
        profile.completeness = this.generateProfileCompleteness(profile);

        // Cache for 5 minutes
        await this.cacheManager.set(cacheKey, profile, 300);
        
        return profile;
    }

    async updateProfile(memberId: string, data: ProfileUpdateDto): Promise<MemberProfile> {
        // Validate input
        const validation = this.validateProfileData(data);
        if (!validation.isValid) {
            throw new Error(`Validation failed: ${validation.errors.join(', ')}`);
        }

        // Update in database
        const updated = await this.memberRepository.update(memberId, data);
        
        // Log the change
        await this.auditLogger.logProfileUpdate(memberId, data);
        
        // Invalidate cache
        await this.invalidateCache(memberId);
        
        // Return updated profile
        return this.getProfile(memberId);
    }

    async uploadAvatar(memberId: string, file: File): Promise<string> {
        // Process image (resize, optimize)
        const processed = await this.processAvatarImage(file);
        
        // Upload to storage
        const key = `avatars/${memberId}/${Date.now()}.jpg`;
        const url = await this.fileStorage.upload(key, processed.buffer);
        
        // Update member record
        await this.memberRepository.update(memberId, { avatarUrl: url });
        
        // Log the change
        await this.auditLogger.logAvatarChange(memberId, 'upload');
        
        // Invalidate cache
        await this.invalidateCache(memberId);
        
        return url;
    }

    private generateProfileCompleteness(profile: MemberProfile): number {
        let score = 0;
        const weights = {
            firstName: 10,
            lastName: 10,
            email: 10,
            phone: 10,
            avatarUrl: 15,
            bio: 15,
            credentials: 20,
            socialLinks: 10
        };

        if (profile.firstName) score += weights.firstName;
        if (profile.lastName) score += weights.lastName;
        if (profile.email) score += weights.email;
        if (profile.phone) score += weights.phone;
        if (profile.avatarUrl) score += weights.avatarUrl;
        if (profile.bio && profile.bio.length > 50) score += weights.bio;
        if (profile.credentials && profile.credentials.length > 0) score += weights.credentials;
        if (profile.socialLinks && profile.socialLinks.length > 0) score += weights.socialLinks;

        return Math.min(score, 100);
    }

    private validateProfileData(data: ProfileUpdateDto): ValidationResult {
        return ProfileValidator.validate(data);
    }

    private async processAvatarImage(file: File): Promise<ProcessedImage> {
        // Implementation for image processing
        // Resize to standard dimensions, optimize file size, etc.
        return processedImage;
    }

    private async invalidateCache(memberId: string): Promise<void> {
        await this.cacheManager.invalidate(`profile:${memberId}`);
        await this.cacheManager.invalidate(`member:${memberId}`);
    }

    private mapToProfile(member: Member): MemberProfile {
        // Map database entity to profile DTO
        return {
            id: member.id,
            firstName: member.firstName,
            lastName: member.lastName,
            email: member.email,
            phone: member.phone,
            avatarUrl: member.avatarUrl,
            bio: member.bio,
            credentials: member.credentials,
            socialLinks: member.socialLinks,
            privacy: member.privacySettings,
            completeness: 0,
            createdAt: member.createdAt,
            updatedAt: member.updatedAt
        };
    }
}
```

## Business Process Flows

### Member Registration and Onboarding Flow

```mermaid
sequenceDiagram
    participant Guest
    participant WebApp
    participant Gateway
    participant AuthService
    participant MemberService
    participant BillingService
    participant NotificationService
    participant CRMService
    
    Guest->>WebApp: Visit registration page
    WebApp->>Guest: Show registration form
    Guest->>WebApp: Submit registration details
    WebApp->>Gateway: POST /api/auth/register
    Gateway->>AuthService: Validate & create account
    
    AuthService->>AuthService: Check duplicate email
    AuthService->>AuthService: Hash password
    AuthService->>AuthService: Generate verification token
    AuthService->>MemberService: Create member profile
    
    MemberService->>MemberService: Create member record
    MemberService->>MemberService: Assign default membership
    MemberService->>BillingService: Create billing account
    
    BillingService->>BillingService: Setup payment profile
    BillingService->>BillingService: Apply initial pricing
    BillingService-->>MemberService: Billing account created
    
    MemberService->>CRMService: Create CRM contact
    CRMService->>CRMService: Create contact record
    CRMService->>CRMService: Trigger welcome workflow
    CRMService-->>MemberService: Contact created
    
    MemberService-->>AuthService: Member created
    AuthService->>NotificationService: Send welcome email
    
    NotificationService->>NotificationService: Render email template
    NotificationService->>NotificationService: Queue email
    NotificationService-->>AuthService: Email queued
    
    AuthService-->>Gateway: Registration successful
    Gateway-->>WebApp: Return success + JWT
    WebApp-->>Guest: Show welcome dashboard
```

### Event Registration with Payment Flow

```mermaid
sequenceDiagram
    participant Member
    participant WebApp
    participant EventService
    participant BillingService
    participant PaymentGateway
    participant NotificationService
    participant CRMService
    
    Member->>WebApp: Browse events
    WebApp->>EventService: GET /api/events
    EventService-->>WebApp: Return event list
    WebApp-->>Member: Display events
    
    Member->>WebApp: Select event & sessions
    WebApp->>EventService: GET /api/events/{id}/availability
    EventService->>EventService: Check capacity
    EventService-->>WebApp: Return availability
    
    Member->>WebApp: Proceed to checkout
    WebApp->>EventService: POST /api/registrations/initiate
    EventService->>EventService: Create pending registration
    EventService->>EventService: Calculate pricing
    EventService->>BillingService: Create invoice
    
    BillingService->>BillingService: Apply member discount
    BillingService->>BillingService: Calculate taxes
    BillingService-->>EventService: Return invoice details
    EventService-->>WebApp: Return checkout data
    
    Member->>WebApp: Enter payment details
    WebApp->>BillingService: POST /api/payments/process
    BillingService->>PaymentGateway: Process payment
    
    PaymentGateway->>PaymentGateway: Validate card
    PaymentGateway->>PaymentGateway: Charge amount
    PaymentGateway-->>BillingService: Payment successful
    
    BillingService->>EventService: Confirm registration
    EventService->>EventService: Update registration status
    EventService->>EventService: Reserve seats
    EventService->>NotificationService: Send confirmation
    
    NotificationService->>NotificationService: Generate email
    NotificationService->>NotificationService: Generate calendar invite
    NotificationService-->>EventService: Notifications sent
    
    EventService->>CRMService: Log activity
    CRMService->>CRMService: Create event registration activity
    CRMService->>CRMService: Update engagement score
    CRMService-->>EventService: Activity logged
    
    EventService-->>WebApp: Registration complete
    WebApp-->>Member: Show confirmation
```

### Certification Management Flow

```mermaid
sequenceDiagram
    participant Member
    participant AdminPortal
    participant ContentService
    participant LMS
    participant CertificationEngine
    participant MemberService
    participant NotificationService
    
    Note over Member,NotificationService: Certification Completion Flow
    
    Member->>ContentService: Access learning content
    ContentService->>LMS: Redirect to course
    LMS-->>Member: Display course content
    
    Member->>LMS: Complete course modules
    LMS->>LMS: Track progress
    Member->>LMS: Take final assessment
    LMS->>LMS: Grade assessment
    
    LMS->>ContentService: POST /webhook/completion
    ContentService->>CertificationEngine: Process completion
    
    CertificationEngine->>CertificationEngine: Verify requirements
    CertificationEngine->>CertificationEngine: Generate certificate
    CertificationEngine->>MemberService: Update member credentials
    
    MemberService->>MemberService: Add certification
    MemberService->>MemberService: Update profile
    MemberService-->>CertificationEngine: Credentials updated
    
    CertificationEngine->>NotificationService: Send certificate
    NotificationService->>NotificationService: Generate PDF
    NotificationService->>NotificationService: Send email
    NotificationService-->>Member: Certificate delivered
    
    Note over AdminPortal,MemberService: Certification Renewal Flow
    
    AdminPortal->>CertificationEngine: Run renewal check
    CertificationEngine->>MemberService: Get expiring certs
    MemberService-->>CertificationEngine: Return list
    
    CertificationEngine->>NotificationService: Send renewal reminders
    NotificationService-->>Member: Renewal notice
    
    Member->>ContentService: Access renewal course
    ContentService->>CertificationEngine: Track renewal progress
```

## Integration Patterns

### API Gateway Pattern

```mermaid
graph TB
    subgraph "Client Applications"
        Web[Web App]
        Mobile[Mobile App]
        Partner[Partner API]
    end
    
    subgraph "API Gateway"
        RateLimit[Rate Limiter<br/>10k req/min per client]
        Auth[Authentication<br/>JWT/OAuth2/API Key]
        Router[Request Router<br/>Path-based routing]
        Transform[Request/Response<br/>Transformer]
        Cache[Response Cache<br/>Redis-backed]
        CircuitBreaker[Circuit Breaker<br/>Fault tolerance]
    end
    
    subgraph "Backend Services"
        Service1[Member Service]
        Service2[Event Service]
        Service3[CRM Service]
        ServiceN[Other Services]
    end
    
    Web --> RateLimit
    Mobile --> RateLimit
    Partner --> RateLimit
    
    RateLimit --> Auth
    Auth --> Router
    Router --> Transform
    Transform --> Cache
    Cache --> CircuitBreaker
    
    CircuitBreaker --> Service1
    CircuitBreaker --> Service2
    CircuitBreaker --> Service3
    CircuitBreaker --> ServiceN
    
    style RateLimit fill:#FF5252,stroke:#C62828
    style Auth fill:#536DFE,stroke:#304FFE
    style CircuitBreaker fill:#FFC107,stroke:#F57C00
```

### Event-Driven Integration Pattern

```mermaid
graph LR
    subgraph "Event Producers"
        MemberSvc[Member Service]
        EventSvc[Event Service]
        BillingSvc[Billing Service]
    end
    
    subgraph "Message Bus"
        Exchange[Topic Exchange<br/>RabbitMQ/Kafka]
        Queue1[member.events]
        Queue2[event.registrations]
        Queue3[billing.transactions]
        DLQ[Dead Letter Queue]
    end
    
    subgraph "Event Consumers"
        NotificationConsumer[Notification Consumer<br/>Email/SMS sender]
        AnalyticsConsumer[Analytics Consumer<br/>Data aggregator]
        IntegrationConsumer[Integration Consumer<br/>External sync]
        AuditConsumer[Audit Consumer<br/>Compliance logger]
    end
    
    MemberSvc -->|Publishes| Exchange
    EventSvc -->|Publishes| Exchange
    BillingSvc -->|Publishes| Exchange
    
    Exchange --> Queue1
    Exchange --> Queue2
    Exchange --> Queue3
    Exchange --> DLQ
    
    Queue1 --> NotificationConsumer
    Queue1 --> AnalyticsConsumer
    Queue2 --> NotificationConsumer
    Queue2 --> IntegrationConsumer
    Queue3 --> AuditConsumer
    Queue3 --> AnalyticsConsumer
    
    NotificationConsumer -.->|Failed messages| DLQ
    IntegrationConsumer -.->|Failed messages| DLQ
```

### Data Synchronization Pattern

```mermaid
graph TB
    subgraph "Association Platform"
        MasterData[Master Data<br/>PostgreSQL]
        ChangeCapture[Change Data Capture<br/>Debezium/CDC]
        SyncOrchestrator[Sync Orchestrator<br/>Apache Airflow]
    end
    
    subgraph "External Systems"
        ERP[ERP System<br/>NetSuite/SAP]
        CRM[External CRM<br/>Salesforce/HubSpot]
        LMS[Learning System<br/>Moodle/Canvas]
    end
    
    subgraph "Sync Infrastructure"
        Queue[Message Queue<br/>Kafka/SQS]
        Transform[Data Transformer<br/>ETL Pipeline]
        ConflictResolver[Conflict Resolver<br/>Last-write-wins]
        SyncLog[Sync Log<br/>Audit trail]
    end
    
    MasterData --> ChangeCapture
    ChangeCapture --> Queue
    Queue --> Transform
    Transform --> ConflictResolver
    ConflictResolver --> SyncOrchestrator
    
    SyncOrchestrator <--> ERP
    SyncOrchestrator <--> CRM
    SyncOrchestrator <--> LMS
    
    SyncOrchestrator --> SyncLog
    ConflictResolver --> SyncLog
    
    style ChangeCapture fill:#4CAF50,stroke:#2E7D32
    style ConflictResolver fill:#FF9800,stroke:#E65100
```

## Security Architecture

### Authentication and Authorization Flow

```mermaid
graph TB
    subgraph "Authentication Layer"
        Login[Login Endpoint]
        OAuth[OAuth2 Provider<br/>Google, LinkedIn]
        SAML[SAML Provider<br/>Okta, Azure AD]
        MFA[MFA Service<br/>TOTP/SMS]
    end
    
    subgraph "Token Management"
        TokenGen[Token Generator<br/>JWT with RSA]
        TokenStore[Token Store<br/>Redis]
        RefreshToken[Refresh Token<br/>Secure storage]
    end
    
    subgraph "Authorization"
        RBAC[Role-Based Access<br/>Roles & Permissions]
        ABAC[Attribute-Based<br/>Context rules]
        PolicyEngine[Policy Engine<br/>OPA/Casbin]
        PermissionCache[Permission Cache<br/>User permissions]
    end
    
    subgraph "Security Services"
        RateLimiter[Rate Limiter<br/>IP/User based]
        IPWhitelist[IP Whitelist<br/>Admin access]
        AuditLog[Security Audit<br/>All actions logged]
        ThreatDetection[Threat Detection<br/>Anomaly detection]
    end
    
    Login --> TokenGen
    OAuth --> TokenGen
    SAML --> TokenGen
    Login --> MFA
    MFA --> TokenGen
    
    TokenGen --> TokenStore
    TokenGen --> RefreshToken
    TokenStore --> RBAC
    
    RBAC --> PolicyEngine
    ABAC --> PolicyEngine
    PolicyEngine --> PermissionCache
    
    Login --> RateLimiter
    RateLimiter --> IPWhitelist
    PolicyEngine --> AuditLog
    AuditLog --> ThreatDetection
    
    style MFA fill:#FF5252,stroke:#C62828
    style PolicyEngine fill:#4CAF50,stroke:#2E7D32
```

### Data Security Layers

```mermaid
graph TB
    subgraph "Application Layer Security"
        InputValidation[Input Validation<br/>XSS/SQLi prevention]
        OutputEncoding[Output Encoding<br/>Context-aware encoding]
        CSRFProtection[CSRF Protection<br/>Token validation]
    end
    
    subgraph "API Security"
        APIAuth[API Authentication<br/>JWT/API Keys]
        APIRateLimit[API Rate Limiting<br/>Tier-based limits]
        APIVersioning[API Versioning<br/>Backward compatibility]
        CORS[CORS Policy<br/>Whitelisted origins]
    end
    
    subgraph "Data Security"
        Encryption[Encryption at Rest<br/>AES-256]
        TLS[TLS 1.3<br/>In transit]
        FieldEncryption[Field Encryption<br/>PII/Sensitive data]
        Tokenization[Tokenization<br/>Payment data]
    end
    
    subgraph "Infrastructure Security"
        WAF[Web Application Firewall<br/>CloudFlare/AWS WAF]
        DDoSProtection[DDoS Protection<br/>Rate limiting]
        NetworkSegmentation[Network Segmentation<br/>VPC/Subnets]
        SecretsManager[Secrets Manager<br/>Vault/AWS Secrets]
    end
    
    InputValidation --> OutputEncoding
    OutputEncoding --> CSRFProtection
    
    APIAuth --> APIRateLimit
    APIRateLimit --> APIVersioning
    APIVersioning --> CORS
    
    Encryption --> FieldEncryption
    TLS --> Tokenization
    
    WAF --> DDoSProtection
    DDoSProtection --> NetworkSegmentation
    NetworkSegmentation --> SecretsManager
    
    style Encryption fill:#1976D2,stroke:#0D47A1
    style WAF fill:#F57C00,stroke:#E65100
```

## Data Architecture

### Multi-Tenant Data Model

```mermaid
erDiagram
    ASSOCIATION {
        uuid id PK
        string name
        string subdomain UK
        string timezone
        json settings
        boolean is_active
        timestamp created_at
        timestamp updated_at
    }
    
    MEMBER {
        uuid id PK
        uuid association_id FK
        string email UK
        string first_name
        string last_name
        string member_number UK
        json profile_data
        timestamp joined_at
        timestamp last_login
    }
    
    MEMBERSHIP {
        uuid id PK
        uuid member_id FK
        uuid membership_type_id FK
        date start_date
        date end_date
        string status
        decimal amount_paid
        timestamp created_at
    }
    
    MEMBERSHIP_TYPE {
        uuid id PK
        uuid association_id FK
        string name
        string code UK
        decimal price
        integer duration_months
        json benefits
        boolean is_active
    }
    
    EVENT {
        uuid id PK
        uuid association_id FK
        string title
        text description
        datetime start_date
        datetime end_date
        string location
        integer capacity
        string status
        json settings
    }
    
    REGISTRATION {
        uuid id PK
        uuid event_id FK
        uuid member_id FK
        string status
        decimal amount_paid
        json registration_data
        timestamp registered_at
    }
    
    CONTENT {
        uuid id PK
        uuid association_id FK
        string title
        string content_type
        text body
        json metadata
        boolean is_public
        timestamp published_at
    }
    
    FORUM_POST {
        uuid id PK
        uuid association_id FK
        uuid member_id FK
        uuid parent_id FK
        string title
        text content
        integer likes_count
        timestamp posted_at
    }
    
    CERTIFICATION {
        uuid id PK
        uuid member_id FK
        uuid certification_type_id FK
        string certificate_number UK
        date issued_date
        date expiry_date
        string status
        json metadata
    }
    
    CRM_CONTACT {
        uuid id PK
        uuid association_id FK
        uuid member_id FK
        string type
        json contact_data
        json custom_fields
        timestamp last_activity
    }
    
    CRM_OPPORTUNITY {
        uuid id PK
        uuid association_id FK
        uuid contact_id FK
        string name
        decimal amount
        string stage
        date close_date
        integer probability
        json metadata
    }
    
    ASSOCIATION ||--o{ MEMBER : has
    ASSOCIATION ||--o{ MEMBERSHIP_TYPE : defines
    ASSOCIATION ||--o{ EVENT : hosts
    ASSOCIATION ||--o{ CONTENT : publishes
    ASSOCIATION ||--o{ FORUM_POST : contains
    ASSOCIATION ||--o{ CRM_CONTACT : manages
    ASSOCIATION ||--o{ CRM_OPPORTUNITY : tracks
    
    MEMBER ||--o{ MEMBERSHIP : has
    MEMBER ||--o{ REGISTRATION : makes
    MEMBER ||--o{ FORUM_POST : creates
    MEMBER ||--o{ CERTIFICATION : earns
    MEMBER ||--|| CRM_CONTACT : "linked to"
    
    MEMBERSHIP_TYPE ||--o{ MEMBERSHIP : "instance of"
    EVENT ||--o{ REGISTRATION : has
    CRM_CONTACT ||--o{ CRM_OPPORTUNITY : has
    FORUM_POST ||--o{ FORUM_POST : replies
```

### Data Partitioning Strategy

```mermaid
graph TB
    subgraph "Data Partitioning"
        subgraph "By Association (Tenant)"
            Tables1[Member Tables<br/>Partitioned by association_id]
            Tables2[Event Tables<br/>Partitioned by association_id]
            Tables3[Content Tables<br/>Partitioned by association_id]
        end
        
        subgraph "By Time"
            Archive1[Registration Archive<br/>Partitioned by year]
            Archive2[Activity Logs<br/>Partitioned by month]
            Archive3[Audit Trail<br/>Partitioned by month]
        end
        
        subgraph "By Size"
            Large1[Forum Posts<br/>Sharded by post_id]
            Large2[File Storage<br/>Distributed by hash]
            Large3[Email Queue<br/>Partitioned by priority]
        end
    end
    
    subgraph "Read Replicas"
        Primary[(Primary DB<br/>PostgreSQL)]
        Replica1[(Read Replica 1<br/>Reporting)]
        Replica2[(Read Replica 2<br/>Analytics)]
        Replica3[(Read Replica 3<br/>Backup)]
    end
    
    subgraph "Caching Layer"
        L1Cache[L1 Cache<br/>Application cache]
        L2Cache[L2 Cache<br/>Redis cluster]
        CDNCache[CDN Cache<br/>Static content]
    end
    
    Primary --> Replica1
    Primary --> Replica2
    Primary --> Replica3
    
    Tables1 --> Primary
    Tables2 --> Primary
    Tables3 --> Primary
    Archive1 --> Primary
    Archive2 --> Primary
    Archive3 --> Primary
    Large1 --> Primary
    Large2 --> Primary
    Large3 --> Primary
    
    Replica1 --> L2Cache
    Replica2 --> L2Cache
    L2Cache --> L1Cache
    CDNCache --> L1Cache
    
    style Primary fill:#4CAF50,stroke:#2E7D32,stroke-width:3px
    style L2Cache fill:#FF5722,stroke:#D32F2F,stroke-width:2px
```

### Analytics Data Pipeline

```mermaid
graph LR
    subgraph "Data Sources"
        AppDB[(Application DB<br/>PostgreSQL)]
        EventStream[Event Stream<br/>Kafka/Kinesis]
        Logs[Application Logs<br/>CloudWatch/Datadog]
        External[External APIs<br/>Payment/Social]
    end
    
    subgraph "ETL Pipeline"
        Extractor[Data Extractor<br/>Apache Airflow]
        Transformer[Transformer<br/>Spark/Databricks]
        Validator[Data Validator<br/>Quality checks]
        Loader[Data Loader<br/>Batch/Stream]
    end
    
    subgraph "Data Warehouse"
        Staging[Staging Area<br/>Raw data]
        DW[Data Warehouse<br/>Snowflake/Redshift]
        DataMart1[Member Analytics<br/>Pre-aggregated]
        DataMart2[Financial Analytics<br/>Revenue/Costs]
        DataMart3[Engagement Analytics<br/>Activity metrics]
    end
    
    subgraph "Analytics Layer"
        OLAP[OLAP Cubes<br/>Fast queries]
        ML[ML Models<br/>Predictions]
        BI[BI Tools<br/>Tableau/PowerBI]
        API[Analytics API<br/>Custom dashboards]
    end
    
    AppDB --> Extractor
    EventStream --> Extractor
    Logs --> Extractor
    External --> Extractor
    
    Extractor --> Transformer
    Transformer --> Validator
    Validator --> Loader
    
    Loader --> Staging
    Staging --> DW
    DW --> DataMart1
    DW --> DataMart2
    DW --> DataMart3
    
    DataMart1 --> OLAP
    DataMart2 --> OLAP
    DataMart3 --> OLAP
    OLAP --> BI
    OLAP --> API
    DW --> ML
    ML --> API
    
    style DW fill:#3F51B5,stroke:#1A237E,stroke-width:3px
    style ML fill:#4CAF50,stroke:#2E7D32,stroke-width:2px
```

## Performance Optimization Strategies

### Caching Strategy

```mermaid
graph TB
    subgraph "Cache Hierarchy"
        Browser[Browser Cache<br/>Static assets]
        CDN[CDN Cache<br/>Global edge locations]
        AppCache[Application Cache<br/>In-memory]
        RedisCache[Redis Cache<br/>Distributed cache]
        DBCache[Database Cache<br/>Query cache]
    end
    
    subgraph "Cache Patterns"
        CacheAside[Cache-Aside<br/>Load on miss]
        WriteThrough[Write-Through<br/>Update cache & DB]
        WriteBehind[Write-Behind<br/>Async DB update]
        RefreshAhead[Refresh-Ahead<br/>Proactive refresh]
    end
    
    subgraph "Cache Keys"
        UserCache[User Sessions<br/>JWT tokens]
        EntityCache[Entity Cache<br/>Members, Events]
        ListCache[List Cache<br/>Search results]
        AggCache[Aggregation Cache<br/>Reports, counts]
    end
    
    Browser --> CDN
    CDN --> AppCache
    AppCache --> RedisCache
    RedisCache --> DBCache
    
    CacheAside --> EntityCache
    WriteThrough --> UserCache
    WriteBehind --> AggCache
    RefreshAhead --> ListCache
    
    style RedisCache fill:#DC382D,stroke:#B71C1C,stroke-width:3px
```

### Load Balancing Architecture

```mermaid
graph TB
    subgraph "Traffic Distribution"
        DNS[DNS Load Balancing<br/>Route 53/CloudFlare]
        GLB[Global Load Balancer<br/>Geographic routing]
        ALB[Application Load Balancer<br/>Layer 7 routing]
        NLB[Network Load Balancer<br/>Layer 4 routing]
    end
    
    subgraph "Application Tier"
        WebCluster[Web Server Cluster<br/>Auto-scaling group]
        APICluster[API Server Cluster<br/>Container orchestration]
        WSCluster[WebSocket Cluster<br/>Sticky sessions]
    end
    
    subgraph "Service Mesh"
        Ingress[Ingress Controller<br/>Traffic management]
        ServiceDiscovery[Service Discovery<br/>Consul/Eureka]
        CircuitBreaker[Circuit Breaker<br/>Fault tolerance]
        Retry[Retry Logic<br/>Exponential backoff]
    end
    
    DNS --> GLB
    GLB --> ALB
    GLB --> NLB
    
    ALB --> WebCluster
    ALB --> APICluster
    NLB --> WSCluster
    
    APICluster --> Ingress
    Ingress --> ServiceDiscovery
    ServiceDiscovery --> CircuitBreaker
    CircuitBreaker --> Retry
    
    style ALB fill:#FF9800,stroke:#E65100,stroke-width:3px
    style ServiceDiscovery fill:#673AB7,stroke:#311B92,stroke-width:2px
```

## Monitoring and Observability

### Observability Stack

```mermaid
graph TB
    subgraph "Data Collection"
        AppMetrics[Application Metrics<br/>Prometheus/StatsD]
        AppLogs[Application Logs<br/>Structured logging]
        AppTraces[Distributed Traces<br/>OpenTelemetry]
        InfraMetrics[Infrastructure Metrics<br/>Node exporters]
    end
    
    subgraph "Data Processing"
        LogAggregator[Log Aggregator<br/>Fluentd/Logstash]
        MetricProcessor[Metric Processor<br/>Prometheus]
        TraceCollector[Trace Collector<br/>Jaeger/Zipkin]
        EventProcessor[Event Processor<br/>Stream processing]
    end
    
    subgraph "Storage"
        TimeSeriesDB[(Time Series DB<br/>Prometheus/InfluxDB)]
        LogStorage[(Log Storage<br/>Elasticsearch)]
        TraceStorage[(Trace Storage<br/>Cassandra)]
        ObjectStorage[(Object Storage<br/>S3/Blob)]
    end
    
    subgraph "Visualization"
        Dashboards[Dashboards<br/>Grafana]
        LogExplorer[Log Explorer<br/>Kibana]
        TraceAnalyzer[Trace Analyzer<br/>Jaeger UI]
        Alerts[Alert Manager<br/>PagerDuty]
    end
    
    AppMetrics --> MetricProcessor
    AppLogs --> LogAggregator
    AppTraces --> TraceCollector
    InfraMetrics --> MetricProcessor
    
    MetricProcessor --> TimeSeriesDB
    LogAggregator --> LogStorage
    TraceCollector --> TraceStorage
    EventProcessor --> ObjectStorage
    
    TimeSeriesDB --> Dashboards
    LogStorage --> LogExplorer
    TraceStorage --> TraceAnalyzer
    TimeSeriesDB --> Alerts
    
    style MetricProcessor fill:#FF6B35,stroke:#D84315,stroke-width:2px
    style Dashboards fill:#00897B,stroke:#00695C,stroke-width:2px
```

## Deployment Architecture

### Container Orchestration

```mermaid
graph TB
    subgraph "Container Registry"
        Registry[Container Registry<br/>ECR/ACR/GCR]
        BaseImages[Base Images<br/>Node, Java, Python]
        AppImages[Application Images<br/>Microservices]
    end
    
    subgraph "Kubernetes Cluster"
        Master[K8s Master<br/>Control plane]
        subgraph "Worker Nodes"
            Node1[Worker Node 1<br/>8 CPU, 32GB RAM]
            Node2[Worker Node 2<br/>8 CPU, 32GB RAM]
            Node3[Worker Node 3<br/>8 CPU, 32GB RAM]
        end
        
        subgraph "Namespaces"
            ProdNS[Production<br/>Live services]
            StagingNS[Staging<br/>Pre-prod testing]
            DevNS[Development<br/>Dev/Test env]
        end
    end
    
    subgraph "K8s Resources"
        Deployments[Deployments<br/>Replica management]
        Services[Services<br/>Load balancing]
        Ingress[Ingress<br/>HTTP routing]
        ConfigMaps[ConfigMaps<br/>Configuration]
        Secrets[Secrets<br/>Sensitive data]
        HPA[HPA<br/>Auto-scaling]
    end
    
    Registry --> Master
    BaseImages --> AppImages
    AppImages --> Registry
    
    Master --> Node1
    Master --> Node2
    Master --> Node3
    
    Node1 --> ProdNS
    Node2 --> ProdNS
    Node3 --> StagingNS
    
    ProdNS --> Deployments
    Deployments --> Services
    Services --> Ingress
    Deployments --> ConfigMaps
    Deployments --> Secrets
    Deployments --> HPA
    
    style Master fill:#1976D2,stroke:#0D47A1,stroke-width:3px
    style ProdNS fill:#388E3C,stroke:#1B5E20,stroke-width:2px
```

### CI/CD Pipeline

```mermaid
graph LR
    subgraph "Version Control"
        Git[Git Repository<br/>GitHub/GitLab]
        Branch[Feature Branch]
        PR[Pull Request]
    end
    
    subgraph "CI Pipeline"
        Build[Build<br/>Compile code]
        Test[Test<br/>Unit/Integration]
        Scan[Security Scan<br/>SAST/DAST]
        Package[Package<br/>Docker images]
    end
    
    subgraph "CD Pipeline"
        Deploy1[Deploy to Dev<br/>Automatic]
        Deploy2[Deploy to Staging<br/>Automatic]
        Deploy3[Deploy to Prod<br/>Manual approval]
        Rollback[Rollback<br/>Previous version]
    end
    
    subgraph "Quality Gates"
        Coverage[Code Coverage<br/>>80%]
        Performance[Performance Test<br/>Load testing]
        Security[Security Gate<br/>Vulnerability check]
        Approval[Manual Approval<br/>Release manager]
    end
    
    Git --> Branch
    Branch --> PR
    PR --> Build
    Build --> Test
    Test --> Scan
    Scan --> Package
    
    Package --> Deploy1
    Deploy1 --> Coverage
    Coverage --> Deploy2
    Deploy2 --> Performance
    Performance --> Security
    Security --> Approval
    Approval --> Deploy3
    Deploy3 -.-> Rollback
    
    style Build fill:#FFC107,stroke:#F57C00,stroke-width:2px
    style Deploy3 fill:#4CAF50,stroke:#2E7D32,stroke-width:3px
```

## Disaster Recovery Plan

### Backup and Recovery Architecture

```mermaid
graph TB
    subgraph "Primary Region"
        PrimaryDB[(Primary Database<br/>PostgreSQL)]
        PrimaryApp[Application Servers<br/>K8s Cluster]
        PrimaryStorage[Object Storage<br/>S3 Bucket]
    end
    
    subgraph "Backup Strategy"
        Snapshot[DB Snapshots<br/>Every 6 hours]
        Continuous[Continuous Backup<br/>WAL streaming]
        AppBackup[Application State<br/>ConfigMaps/Secrets]
        DataExport[Data Export<br/>Daily full export]
    end
    
    subgraph "Secondary Region"
        StandbyDB[(Standby Database<br/>Read replica)]
        StandbyApp[Standby Servers<br/>Inactive K8s]
        StandbyStorage[Replicated Storage<br/>Cross-region]
    end
    
    subgraph "Recovery Process"
        Detection[Failure Detection<br/>Health checks]
        Failover[Automated Failover<br/>DNS switch]
        Validation[Validation<br/>Data integrity]
        Notification[Notification<br/>Alert teams]
    end
    
    PrimaryDB --> Snapshot
    PrimaryDB --> Continuous
    PrimaryApp --> AppBackup
    PrimaryDB --> DataExport
    
    Continuous --> StandbyDB
    AppBackup --> StandbyApp
    PrimaryStorage --> StandbyStorage
    
    Detection --> Failover
    Failover --> StandbyDB
    Failover --> StandbyApp
    Failover --> Validation
    Validation --> Notification
    
    style PrimaryDB fill:#1976D2,stroke:#0D47A1,stroke-width:3px
    style Failover fill:#FF5252,stroke:#C62828,stroke-width:3px
```

## Conclusion

This C4 architecture documentation provides a comprehensive view of the Association Management Platform, from high-level system context down to detailed code structure. The architecture supports:

1. **Scalability**: Multi-tenant architecture with horizontal scaling capabilities
2. **Reliability**: Redundant systems with automated failover
3. **Security**: Multiple layers of security from application to infrastructure
4. **Performance**: Caching strategies and load balancing for optimal performance
5. **Maintainability**: Clear separation of concerns with microservices architecture
6. **Integration**: Flexible integration patterns for external systems

The platform is designed to handle the complex needs of professional associations while providing a modern, scalable, and secure foundation for growth.