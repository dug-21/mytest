# Authentication and SSO Architecture

## Overview

This document outlines a comprehensive authentication and Single Sign-On (SSO) architecture that provides secure, seamless access across legacy systems, modern CRM platforms, and external services while maintaining compliance with industry standards.

## SSO Architecture Overview

```mermaid
graph TB
    subgraph "User Access Points"
        BROWSER[Web Browser]
        MOBILE[Mobile App]
        API_CLIENT[API Client]
        PARTNER[Partner Portal]
    end
    
    subgraph "Identity Provider (IdP)"
        IDP[Identity Provider<br/>Okta/Auth0/Keycloak]
        USER_STORE[User Store<br/>LDAP/AD]
        MFA[Multi-Factor Auth<br/>TOTP/SMS/Push]
        CONSENT[Consent Management]
    end
    
    subgraph "Authentication Services"
        AUTH_PROXY[Auth Proxy]
        TOKEN_SVC[Token Service]
        SESSION_MGR[Session Manager]
        DEVICE_MGR[Device Registry]
    end
    
    subgraph "Protocol Support"
        OAUTH2[OAuth 2.0]
        SAML[SAML 2.0]
        OIDC[OpenID Connect]
        JWT_HANDLER[JWT Handler]
    end
    
    subgraph "Service Providers (SP)"
        LEGACY_SP[Legacy Platform]
        CRM_SP[CRM System]
        LMS_SP[LMS Platform]
        EMAIL_SP[Email System]
    end
    
    subgraph "Security Layer"
        RISK_ENGINE[Risk Assessment]
        AUDIT_LOG[Audit Logger]
        ANOMALY[Anomaly Detection]
        COMPLIANCE[Compliance Engine]
    end
    
    %% User flows
    BROWSER --> AUTH_PROXY
    MOBILE --> AUTH_PROXY
    API_CLIENT --> AUTH_PROXY
    PARTNER --> AUTH_PROXY
    
    %% Auth proxy to IdP
    AUTH_PROXY --> IDP
    IDP --> USER_STORE
    IDP --> MFA
    IDP --> CONSENT
    
    %% Protocol handling
    IDP --> OAUTH2
    IDP --> SAML
    IDP --> OIDC
    
    %% Token management
    OAUTH2 --> TOKEN_SVC
    OIDC --> TOKEN_SVC
    TOKEN_SVC --> JWT_HANDLER
    TOKEN_SVC --> SESSION_MGR
    
    %% Service provider connections
    JWT_HANDLER --> LEGACY_SP
    SAML --> CRM_SP
    OIDC --> LMS_SP
    OAUTH2 --> EMAIL_SP
    
    %% Security monitoring
    AUTH_PROXY --> RISK_ENGINE
    TOKEN_SVC --> AUDIT_LOG
    SESSION_MGR --> ANOMALY
    AUDIT_LOG --> COMPLIANCE
    
    %% Device management
    AUTH_PROXY --> DEVICE_MGR
    DEVICE_MGR --> RISK_ENGINE
    
    classDef security fill:#ffebee
    classDef auth fill:#e3f2fd
    classDef service fill:#e8f5e9
    
    class RISK_ENGINE,AUDIT_LOG,ANOMALY,COMPLIANCE security
    class IDP,AUTH_PROXY,TOKEN_SVC,SESSION_MGR auth
    class LEGACY_SP,CRM_SP,LMS_SP,EMAIL_SP service
```

## Authentication Flows

### 1. Web Browser SSO Flow (SAML)

```mermaid
sequenceDiagram
    participant User
    participant Browser
    participant SP as Service Provider
    participant IdP as Identity Provider
    participant MFA as MFA Service
    participant Audit as Audit Service
    
    User->>Browser: Access protected resource
    Browser->>SP: GET /protected-resource
    
    alt No valid session
        SP->>Browser: Redirect to IdP
        Browser->>IdP: SAML AuthnRequest
        IdP->>Browser: Login page
        User->>Browser: Enter credentials
        Browser->>IdP: POST credentials
        
        IdP->>IdP: Validate credentials
        IdP->>MFA: Trigger MFA
        MFA->>User: Send MFA challenge
        User->>MFA: Complete MFA
        MFA->>IdP: MFA success
        
        IdP->>Audit: Log authentication event
        IdP->>Browser: SAML Response (encrypted)
        Browser->>SP: POST SAML Response
        SP->>SP: Validate SAML assertion
        SP->>Browser: Set session cookie
        SP->>Browser: Redirect to resource
    else Valid session exists
        SP->>Browser: Return protected resource
    end
    
    Browser->>User: Display resource
```

### 2. Mobile App OAuth 2.0 + PKCE Flow

```mermaid
sequenceDiagram
    participant App as Mobile App
    participant Browser as System Browser
    participant AS as Auth Server
    participant API as API Gateway
    participant RS as Resource Server
    
    App->>App: Generate code_verifier & code_challenge
    App->>Browser: Open auth URL with code_challenge
    Browser->>AS: GET /authorize?code_challenge=...
    AS->>Browser: Show login page
    Browser->>AS: POST credentials
    AS->>Browser: Redirect with auth code
    Browser->>App: Deep link with code
    
    App->>AS: POST /token with code & code_verifier
    AS->>AS: Verify code_challenge
    AS->>App: Return tokens (access & refresh)
    
    App->>API: API request with access token
    API->>API: Validate token
    API->>RS: Forward request
    RS->>API: Response
    API->>App: Return data
    
    Note over App,AS: Token refresh flow when access token expires
```

### 3. API Client Authentication (Client Credentials)

```mermaid
sequenceDiagram
    participant Client as API Client
    participant Gateway as API Gateway
    participant AS as Auth Server
    participant Cache as Token Cache
    participant Service as Backend Service
    
    Client->>AS: POST /token (client_credentials grant)
    Note over Client,AS: Client ID + Client Secret
    
    AS->>AS: Validate client credentials
    AS->>AS: Check client permissions
    AS->>Client: Return access token
    
    Client->>Gateway: API request with Bearer token
    Gateway->>Cache: Check token cache
    
    alt Token in cache
        Cache->>Gateway: Return cached validation
    else Token not cached
        Gateway->>AS: Introspect token
        AS->>Gateway: Token metadata
        Gateway->>Cache: Cache validation
    end
    
    Gateway->>Service: Forward authenticated request
    Service->>Gateway: Response
    Gateway->>Client: Return response
```

## Token Management

### 1. JWT Token Structure

```json
{
  "header": {
    "alg": "RS256",
    "typ": "JWT",
    "kid": "rsa-key-2024-01"
  },
  "payload": {
    "iss": "https://auth.association.org",
    "sub": "user:12345",
    "aud": ["api.association.org", "crm.association.org"],
    "exp": 1706620800,
    "iat": 1706617200,
    "nbf": 1706617200,
    "jti": "token-unique-id-123",
    "auth_time": 1706617200,
    "acr": "mfa",
    "amr": ["pwd", "otp"],
    "scope": "openid profile email members:read members:write",
    "azp": "web-client-id",
    "session_state": "session-abc123",
    "custom_claims": {
      "org_id": "org-456",
      "membership_level": "professional",
      "roles": ["member", "chapter_admin"],
      "permissions": ["view_members", "edit_members", "view_reports"]
    }
  },
  "signature": "..."
}
```

### 2. Token Lifecycle Management

```mermaid
stateDiagram-v2
    [*] --> Issued: Client authenticates
    Issued --> Active: Token validated
    Active --> Refreshing: Near expiration
    Refreshing --> Active: New token issued
    Active --> Expired: Lifetime exceeded
    Refreshing --> Expired: Refresh failed
    Active --> Revoked: User action/Security event
    Expired --> [*]
    Revoked --> [*]
    
    note right of Active
        Regular validation
        Permission checks
        Usage tracking
    end note
    
    note right of Refreshing
        Sliding window
        Rotation policy
        Audit trail
    end note
```

### 3. Token Security Configuration

```yaml
# Token Service Configuration
token_service:
  jwt:
    signing:
      algorithm: RS256
      key_rotation_interval: 30d
      key_overlap_period: 7d
    
    validation:
      verify_signature: true
      verify_expiration: true
      verify_not_before: true
      verify_issuer: true
      verify_audience: true
      clock_skew_seconds: 30
    
    expiration:
      access_token: 3600  # 1 hour
      refresh_token: 2592000  # 30 days
      id_token: 3600  # 1 hour
      session_absolute: 43200  # 12 hours
      session_idle: 7200  # 2 hours
    
    security:
      require_jti: true  # Unique token ID
      store_jti_for_revocation: true
      encrypt_at_rest: true
      token_binding: optional
      dpop_required: false

session_management:
  storage:
    type: redis_cluster
    ttl: 43200  # 12 hours
    encryption: aes-256-gcm
  
  security:
    secure_cookie: true
    http_only: true
    same_site: strict
    domain: .association.org
    
  tracking:
    ip_validation: soft  # warn on change
    user_agent_validation: true
    device_fingerprint: true
```

## Multi-Factor Authentication (MFA)

### 1. MFA Architecture

```mermaid
graph TB
    subgraph "MFA Methods"
        TOTP[TOTP<br/>Google Auth]
        SMS[SMS<br/>Twilio]
        PUSH[Push Notification<br/>Mobile App]
        FIDO[FIDO2/WebAuthn<br/>Security Keys]
        BACKUP[Backup Codes]
    end
    
    subgraph "MFA Engine"
        POLICY[Policy Engine]
        RISK[Risk Scorer]
        SELECTOR[Method Selector]
        VERIFIER[Verifier]
    end
    
    subgraph "Risk Factors"
        IP_CHECK[IP Reputation]
        GEO[Geolocation]
        DEVICE[Device Trust]
        BEHAVIOR[User Behavior]
    end
    
    subgraph "Enforcement"
        ENFORCE[MFA Enforcer]
        BYPASS[Bypass Rules]
        STEP_UP[Step-up Auth]
        REMEMBER[Remember Device]
    end
    
    %% Risk assessment
    IP_CHECK --> RISK
    GEO --> RISK
    DEVICE --> RISK
    BEHAVIOR --> RISK
    
    %% MFA flow
    RISK --> POLICY
    POLICY --> SELECTOR
    SELECTOR --> TOTP
    SELECTOR --> SMS
    SELECTOR --> PUSH
    SELECTOR --> FIDO
    SELECTOR --> BACKUP
    
    %% Verification
    TOTP --> VERIFIER
    SMS --> VERIFIER
    PUSH --> VERIFIER
    FIDO --> VERIFIER
    BACKUP --> VERIFIER
    
    %% Enforcement
    VERIFIER --> ENFORCE
    POLICY --> BYPASS
    ENFORCE --> STEP_UP
    ENFORCE --> REMEMBER
```

### 2. Adaptive MFA Rules

```python
class AdaptiveMFAPolicy:
    def evaluate_risk(self, context):
        """Evaluate authentication risk and determine MFA requirement"""
        risk_score = 0
        mfa_required = False
        mfa_methods = []
        
        # IP reputation check
        if self.is_suspicious_ip(context.ip_address):
            risk_score += 30
            
        # Geolocation check
        if self.is_unusual_location(context.user_id, context.geo_location):
            risk_score += 25
            
        # Device trust check
        if not self.is_trusted_device(context.device_id):
            risk_score += 20
            
        # Time-based check
        if self.is_unusual_time(context.user_id, context.timestamp):
            risk_score += 15
            
        # Behavior analysis
        if self.detect_anomaly(context.user_id, context.behavior_pattern):
            risk_score += 40
            
        # Determine MFA requirement
        if risk_score >= 30:
            mfa_required = True
            
        # Select MFA methods based on risk
        if risk_score >= 70:
            mfa_methods = ['fido2', 'push']  # High security
        elif risk_score >= 40:
            mfa_methods = ['totp', 'push', 'sms']  # Medium security
        else:
            mfa_methods = ['totp', 'sms', 'backup']  # Standard security
            
        return {
            'risk_score': risk_score,
            'mfa_required': mfa_required,
            'allowed_methods': mfa_methods,
            'step_up_after': 3600 if risk_score >= 50 else None
        }
```

## Directory Integration

### 1. LDAP/AD Integration Architecture

```mermaid
graph LR
    subgraph "Identity Sources"
        AD[Active Directory]
        LDAP[OpenLDAP]
        HR[HR System]
        LEGACY_USER[Legacy User DB]
    end
    
    subgraph "Sync Service"
        CONNECTOR[AD/LDAP Connector]
        MAPPER[Attribute Mapper]
        TRANSFORM[Data Transform]
        SCHEDULER[Sync Scheduler]
    end
    
    subgraph "Identity Store"
        IDP_STORE[IdP User Store]
        CACHE[User Cache]
        PROFILE[User Profiles]
        GROUPS[Group Memberships]
    end
    
    subgraph "Provisioning"
        SCIM[SCIM 2.0 API]
        JUST_IN_TIME[JIT Provisioning]
        LIFECYCLE[Lifecycle Manager]
    end
    
    %% Sync flow
    AD --> CONNECTOR
    LDAP --> CONNECTOR
    HR --> CONNECTOR
    LEGACY_USER --> CONNECTOR
    
    CONNECTOR --> MAPPER
    MAPPER --> TRANSFORM
    TRANSFORM --> IDP_STORE
    
    %% Caching
    IDP_STORE --> CACHE
    IDP_STORE --> PROFILE
    IDP_STORE --> GROUPS
    
    %% Provisioning
    SCHEDULER --> CONNECTOR
    IDP_STORE --> SCIM
    IDP_STORE --> JUST_IN_TIME
    SCIM --> LIFECYCLE
```

### 2. Attribute Mapping Configuration

```yaml
# LDAP to IdP Attribute Mapping
attribute_mapping:
  ldap_to_idp:
    # Basic attributes
    uid: username
    cn: display_name
    givenName: first_name
    sn: last_name
    mail: email
    telephoneNumber: phone
    
    # Organization attributes
    o: organization
    ou: department
    title: job_title
    employeeNumber: employee_id
    
    # Custom attributes
    membershipLevel: custom.membership_level
    chapterAffiliation: custom.chapter
    certifications: custom.certifications[]
    
  # Group mappings
  group_mapping:
    "CN=Members,OU=Groups,DC=association,DC=org":
      idp_group: "members"
      permissions: ["basic_access"]
      
    "CN=ChapterAdmins,OU=Groups,DC=association,DC=org":
      idp_group: "chapter_admins"
      permissions: ["chapter_management", "member_view"]
      
    "CN=BoardMembers,OU=Groups,DC=association,DC=org":
      idp_group: "board_members"
      permissions: ["full_access", "financial_reports"]

# Sync configuration
sync_config:
  schedule: "0 */4 * * *"  # Every 4 hours
  batch_size: 1000
  conflict_resolution: "source_wins"
  soft_delete: true
  password_sync: false  # Use SSO instead
```

## Security Considerations

### 1. OAuth Security Profile

```yaml
oauth_security_profile:
  # PKCE is mandatory for public clients
  require_pkce: true
  pkce_challenge_methods: ["S256"]
  
  # Token binding
  token_binding:
    required: true
    methods: ["tls_client_cert", "dpop"]
  
  # Grant types
  allowed_grants:
    - authorization_code
    - refresh_token
    - client_credentials
  
  forbidden_grants:
    - implicit  # Deprecated
    - password  # Not recommended
  
  # Response types
  allowed_response_types:
    - code
  
  # Security headers
  security_headers:
    X-Frame-Options: DENY
    X-Content-Type-Options: nosniff
    X-XSS-Protection: "1; mode=block"
    Strict-Transport-Security: "max-age=31536000; includeSubDomains"
    Content-Security-Policy: "default-src 'self'"
```

### 2. Session Security

```python
class SecureSessionManager:
    def create_session(self, user_id, context):
        """Create secure session with context binding"""
        session = {
            'id': self.generate_secure_id(),
            'user_id': user_id,
            'created_at': datetime.utcnow(),
            'expires_at': datetime.utcnow() + timedelta(hours=12),
            'ip_address': context.ip_address,
            'user_agent': context.user_agent,
            'device_fingerprint': self.calculate_fingerprint(context),
            'security_context': {
                'auth_method': context.auth_method,
                'auth_level': context.auth_level,
                'mfa_satisfied': context.mfa_completed
            }
        }
        
        # Encrypt session data
        encrypted_session = self.encrypt_session(session)
        
        # Store in distributed cache
        self.redis_client.setex(
            f"session:{session['id']}",
            43200,  # 12 hours
            encrypted_session
        )
        
        # Create secure cookie
        cookie_value = self.create_signed_cookie(session['id'])
        
        return {
            'session_id': session['id'],
            'cookie': {
                'value': cookie_value,
                'secure': True,
                'httpOnly': True,
                'sameSite': 'Strict',
                'domain': '.association.org',
                'path': '/',
                'expires': session['expires_at']
            }
        }
    
    def validate_session(self, session_id, context):
        """Validate session with security checks"""
        session = self.get_session(session_id)
        
        if not session:
            raise InvalidSessionError("Session not found")
            
        # Check expiration
        if session['expires_at'] < datetime.utcnow():
            raise SessionExpiredError("Session expired")
            
        # Validate context binding
        if self.strict_validation:
            if session['ip_address'] != context.ip_address:
                self.log_security_event("IP_MISMATCH", session_id)
                if not self.allow_ip_change:
                    raise SecurityError("IP address mismatch")
                    
            if session['user_agent'] != context.user_agent:
                self.log_security_event("USER_AGENT_MISMATCH", session_id)
                raise SecurityError("User agent mismatch")
        
        # Update last activity
        session['last_activity'] = datetime.utcnow()
        self.update_session(session)
        
        return session
```

### 3. Audit and Compliance

```json
{
  "audit_event": {
    "event_id": "evt_2024012015304512345",
    "timestamp": "2024-01-20T15:30:45.123Z",
    "event_type": "authentication.success",
    "actor": {
      "user_id": "user_12345",
      "ip_address": "192.168.1.100",
      "user_agent": "Mozilla/5.0...",
      "session_id": "sess_abc123"
    },
    "authentication": {
      "method": "saml",
      "mfa_used": true,
      "mfa_type": "totp",
      "idp": "okta"
    },
    "target": {
      "service": "crm_system",
      "resource": "/api/members",
      "action": "read"
    },
    "outcome": {
      "result": "success",
      "reason": null
    },
    "metadata": {
      "risk_score": 15,
      "geo_location": "US-CA",
      "device_trusted": true
    }
  }
}
```

## Implementation Checklist

### Phase 1: Foundation
- [ ] Select and deploy Identity Provider (IdP)
- [ ] Configure LDAP/AD integration
- [ ] Set up OAuth 2.0/OIDC endpoints
- [ ] Implement JWT token service
- [ ] Deploy session management

### Phase 2: Integration
- [ ] Configure SAML for legacy system
- [ ] Implement OAuth for modern apps
- [ ] Set up API authentication
- [ ] Configure service provider metadata
- [ ] Test SSO flows

### Phase 3: Security Enhancement
- [ ] Deploy MFA solution
- [ ] Implement adaptive authentication
- [ ] Configure audit logging
- [ ] Set up anomaly detection
- [ ] Enable session security

### Phase 4: Operations
- [ ] Configure monitoring and alerts
- [ ] Set up backup authentication
- [ ] Document runbooks
- [ ] Train support team
- [ ] Plan disaster recovery