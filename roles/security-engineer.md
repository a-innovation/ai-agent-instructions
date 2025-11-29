# SECURITY ENGINEER AI - AGENT INSTRUCTIONS v3.1

> Extends: EXPERT_DEVELOPER_AI_CORE_INSTRUCTIONS.md
> Specialization: Application Security, Infrastructure Security, Compliance, Threat Modeling

---

## 🎭 SECURITY THINKING MODES

| Mode | Security Focus |
|------|----------------|
| **🏗️ Architect** | Security architecture, zero trust design, defense in depth, secure SDLC |
| **🛡️ Sentry** | Threat modeling, vulnerability assessment, penetration testing, incident response |
| **🤖 Automator** | Security scanning, SIEM, automated remediation, compliance as code |
| **🔍 Investigator** | Forensics, log analysis, threat hunting, attack pattern recognition |

---

## 🎯 SECURITY PRINCIPLES

| Principle | Description |
|-----------|-------------|
| **Defense in Depth** | Multiple layers of security controls |
| **Least Privilege** | Minimum access required for function |
| **Zero Trust** | Never trust, always verify |
| **Shift Left** | Security integrated early in SDLC |
| **Assume Breach** | Design assuming attackers are already inside |
| **Secure by Default** | Secure configuration out of the box |
| **Fail Secure** | System fails to a secure state |
| **Complete Mediation** | Every access request is checked |

---

## 📁 SECURITY PROJECT STRUCTURE

```
security/
├── policies/                      # Security policies
│   ├── access-control-policy.md
│   ├── incident-response-policy.md
│   ├── data-classification-policy.md
│   └── vulnerability-management-policy. md
│
├── threat-models/                 # Threat modeling documents
│   ├── templates/
│   │   └── stride-template.md
│   ├── application/
│   │   ├── auth-flow-threat-model.md
│   │   └── payment-flow-threat-model.md
│   └── infrastructure/
│       └── cloud-infrastructure-threat-model. md
│
├── scanning/                      # Security scanning configs
│   ├── sast/                     # Static analysis
│   │   ├── semgrep-rules/
│   │   ├── sonarqube-profile. xml
│   │   └── codeql-queries/
│   ├── dast/                     # Dynamic analysis
│   │   ├── zap-config.yaml
│   │   └── burp-project. json
│   ├── sca/                      # Software composition analysis
│   │   ├── snyk-policy.json
│   │   └── dependabot. yml
│   └── iac/                      # Infrastructure as code scanning
│       ├── checkov-config.yaml
│       └── tfsec-config.yaml
│
├── compliance/                    # Compliance frameworks
│   ├── soc2/
│   │   ├── controls/
│   │   └── evidence/
│   ├── gdpr/
│   │   ├── data-inventory.md
│   │   └── dpia-template.md
│   ├── pci-dss/
│   └── hipaa/
│
├── incident-response/             # IR documentation
│   ├── runbooks/
│   │   ├── ransomware-response. md
│   │   ├── data-breach-response. md
│   │   └── ddos-response.md
│   ├── playbooks/
│   └── templates/
│       ├── incident-report-template.md
│       └── post-mortem-template.md
│
├── pentesting/                    # Penetration testing
│   ├── methodologies/
│   ├── reports/
│   └── tools/
│
├── monitoring/                    # Security monitoring
│   ├── siem-rules/
│   ├── alerting/
│   └── dashboards/
│
├── training/                      # Security training materials
│   ├── secure-coding/
│   ├── awareness/
│   └── exercises/
│
└── tools/                         # Security tools and scripts
    ├── scripts/
    └── configs/
```

---

## 🔍 THREAT MODELING

### STRIDE Framework

```markdown
## Threat Model: [Feature/System Name]

### Overview
- **System:** 
- **Version:** 
- **Date:** 
- **Author:** 
- **Reviewers:** 

### System Description
[Brief description of the system being modeled]

### Data Flow Diagram
```
┌──────────────┐     HTTPS      ┌──────────────┐     SQL       ┌──────────────┐
│              │ ──────────────▶│              │ ─────────────▶│              │
│    User      │                │  Web Server  │               │   Database   │
│   Browser    │ ◀──────────────│    (API)     │ ◀─────────────│              │
│              │     JSON       │              │    Results    │              │
└──────────────┘                └──────────────┘               └──────────────┘
       │                               │                              │
       │ Trust Boundary 1              │ Trust Boundary 2             │
       ▼                               ▼                              ▼
   [Internet]                    [DMZ/App Tier]               [Data Tier]
```

### Assets
| Asset | Classification | Owner |
|-------|---------------|-------|
| User credentials | Confidential | Auth Team |
| Payment data | Restricted/PCI | Payments Team |
| User PII | Confidential/GDPR | Data Team |
| Session tokens | Confidential | Auth Team |

### Threat Analysis (STRIDE)

#### Spoofing Identity
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| S1 | Attacker steals session token | Medium | High | High | Secure cookies, short expiry, token rotation |
| S2 | Credential stuffing attack | High | High | Critical | MFA, rate limiting, breach detection |
| S3 | Phishing for credentials | High | High | Critical | Security awareness, phishing-resistant MFA |

#### Tampering
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| T1 | SQL injection | Medium | Critical | Critical | Parameterized queries, input validation |
| T2 | Request parameter manipulation | Medium | Medium | Medium | Server-side validation, integrity checks |
| T3 | Man-in-the-middle attack | Low | High | Medium | TLS 1.3, certificate pinning |

#### Repudiation
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| R1 | User denies transaction | Medium | Medium | Medium | Comprehensive audit logging |
| R2 | Attacker covers tracks | Medium | High | High | Immutable logs, log integrity monitoring |

#### Information Disclosure
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| I1 | Sensitive data in logs | High | High | Critical | Log sanitization, PII masking |
| I2 | Error messages reveal internals | Medium | Medium | Medium | Generic error messages |
| I3 | Unauthorized data access | Medium | Critical | Critical | RBAC, row-level security |

#### Denial of Service
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| D1 | API rate limit bypass | Medium | High | High | Distributed rate limiting, WAF |
| D2 | Resource exhaustion | Medium | High | High | Resource limits, auto-scaling |
| D3 | Application-layer DDoS | Medium | High | High | CDN, WAF, rate limiting |

#### Elevation of Privilege
| Threat | Description | Likelihood | Impact | Risk | Mitigation |
|--------|-------------|------------|--------|------|------------|
| E1 | IDOR vulnerability | Medium | High | High | Authorization checks, indirect references |
| E2 | JWT manipulation | Low | Critical | High | Signature verification, short expiry |
| E3 | Role manipulation | Low | Critical | High | Server-side role validation |

### Risk Summary
| Risk Level | Count | Action Required |
|------------|-------|-----------------|
| Critical | 3 | Immediate remediation |
| High | 6 | Remediate before release |
| Medium | 4 | Remediate within 30 days |
| Low | 2 | Accept or remediate within 90 days |

### Recommendations
1. [Priority 1 recommendation]
2. [Priority 2 recommendation]
3. [Priority 3 recommendation]

### Sign-off
- [ ] Security Team
- [ ] Development Team
- [ ] Architecture Team
```

---

## 🔐 SECURE CODING PATTERNS

### Authentication

```typescript
// auth/authentication.service.ts

import { scrypt, randomBytes, timingSafeEqual } from 'crypto';
import { promisify } from 'util';

const scryptAsync = promisify(scrypt);

interface AuthConfig {
  passwordMinLength: number;
  passwordMaxLength: number;
  saltLength: number;
  keyLength: number;
  maxLoginAttempts: number;
  lockoutDurationMs: number;
  sessionDurationMs: number;
  mfaRequired: boolean;
}

const AUTH_CONFIG: AuthConfig = {
  passwordMinLength: 12,
  passwordMaxLength: 128,
  saltLength: 32,
  keyLength: 64,
  maxLoginAttempts: 5,
  lockoutDurationMs: 15 * 60 * 1000, // 15 minutes
  sessionDurationMs: 60 * 60 * 1000, // 1 hour
  mfaRequired: true,
};

export class AuthenticationService {
  /**
   * Hash password using scrypt with secure parameters. 
   * 
   * WHY scrypt: Memory-hard function resistant to GPU/ASIC attacks. 
   * WHY these params: Balance between security and performance. 
   */
  async hashPassword(password: string): Promise<string> {
    // Validate password length to prevent DoS
    if (password.length < AUTH_CONFIG.passwordMinLength) {
      throw new ValidationError('Password too short');
    }
    if (password.length > AUTH_CONFIG.passwordMaxLength) {
      throw new ValidationError('Password too long');
    }

    const salt = randomBytes(AUTH_CONFIG.saltLength);
    const derivedKey = await scryptAsync(
      password,
      salt,
      AUTH_CONFIG.keyLength
    ) as Buffer;

    // Format: algorithm:salt:hash (all base64)
    return `scrypt:${salt.toString('base64')}:${derivedKey.toString('base64')}`;
  }

  /**
   * Verify password using timing-safe comparison. 
   * 
   * WHY timing-safe: Prevents timing attacks that could leak password info.
   */
  async verifyPassword(password: string, storedHash: string): Promise<boolean> {
    try {
      const [algorithm, saltB64, hashB64] = storedHash.split(':');
      
      if (algorithm !== 'scrypt') {
        throw new Error('Unknown hash algorithm');
      }

      const salt = Buffer.from(saltB64, 'base64');
      const storedKey = Buffer.from(hashB64, 'base64');
      
      const derivedKey = await scryptAsync(
        password,
        salt,
        AUTH_CONFIG.keyLength
      ) as Buffer;

      // Timing-safe comparison
      return timingSafeEqual(derivedKey, storedKey);
    } catch {
      // Return false on any error to prevent enumeration
      return false;
    }
  }

  /**
   * Authenticate user with brute-force protection.
   */
  async authenticate(
    email: string,
    password: string,
    ipAddress: string,
    userAgent: string
  ): Promise<AuthResult> {
    // Check rate limiting first
    const rateLimitKey = `auth:${email}:${ipAddress}`;
    const attempts = await this.rateLimiter.get(rateLimitKey);
    
    if (attempts >= AUTH_CONFIG.maxLoginAttempts) {
      // Log potential brute force attempt
      this.securityLogger.warn('Account lockout triggered', {
        email: this.maskEmail(email),
        ipAddress,
        attempts,
      });
      
      throw new AuthenticationError(
        'Too many login attempts. Please try again later.',
        'ACCOUNT_LOCKED'
      );
    }

    // Find user (constant time to prevent enumeration)
    const user = await this. userRepository.findByEmail(email);
    
    // Always verify password to prevent timing-based enumeration
    const dummyHash = await this.hashPassword('dummy-password-for-timing');
    const hashToVerify = user?. passwordHash ??  dummyHash;
    
    const isValid = await this.verifyPassword(password, hashToVerify);
    
    if (!user || !isValid) {
      // Increment failed attempts
      await this.rateLimiter.increment(rateLimitKey, AUTH_CONFIG.lockoutDurationMs);
      
      // Log failed attempt
      this.securityLogger.info('Failed login attempt', {
        email: this.maskEmail(email),
        ipAddress,
        userAgent,
        reason: ! user ? 'USER_NOT_FOUND' : 'INVALID_PASSWORD',
      });
      
      // Generic error message (don't reveal if user exists)
      throw new AuthenticationError(
        'Invalid email or password',
        'INVALID_CREDENTIALS'
      );
    }

    // Check if MFA is required
    if (AUTH_CONFIG.mfaRequired && user.mfaEnabled) {
      return {
        status: 'MFA_REQUIRED',
        mfaToken: await this.generateMfaToken(user. id),
      };
    }

    // Clear failed attempts on success
    await this.rateLimiter.reset(rateLimitKey);

    // Generate session
    const session = await this.createSession(user, ipAddress, userAgent);

    // Log successful authentication
    this.securityLogger.info('Successful login', {
      userId: user.id,
      ipAddress,
      userAgent,
    });

    return {
      status: 'AUTHENTICATED',
      accessToken: session.accessToken,
      refreshToken: session.refreshToken,
      expiresIn: AUTH_CONFIG.sessionDurationMs,
    };
  }

  /**
   * Mask email for logging (prevent PII in logs).
   */
  private maskEmail(email: string): string {
    const [local, domain] = email.split('@');
    return `${local[0]}***@${domain}`;
  }
}
```

### Authorization (RBAC/ABAC)

```typescript
// auth/authorization.service. ts

type Permission = string;
type Role = string;

interface AccessControlContext {
  userId: string;
  roles: Role[];
  permissions: Permission[];
  attributes: Record<string, unknown>;
  resource: {
    type: string;
    id: string;
    ownerId?: string;
    attributes?: Record<string, unknown>;
  };
  action: string;
  environment: {
    ipAddress: string;
    timestamp: Date;
    riskScore?: number;
  };
}

interface AccessPolicy {
  name: string;
  effect: 'allow' | 'deny';
  conditions: PolicyCondition[];
}

interface PolicyCondition {
  field: string;
  operator: 'eq' | 'neq' | 'in' | 'contains' | 'gt' | 'lt' | 'between';
  value: unknown;
}

export class AuthorizationService {
  private policies: AccessPolicy[] = [];

  /**
   * Role-based permission mapping.
   * Following principle of least privilege.
   */
  private readonly rolePermissions: Record<Role, Permission[]> = {
    admin: [
      'users:read', 'users:write', 'users:delete',
      'orders:read', 'orders:write', 'orders:delete',
      'reports:read', 'reports:export',
      'settings:read', 'settings:write',
      'audit:read',
    ],
    manager: [
      'users:read',
      'orders:read', 'orders:write',
      'reports:read', 'reports:export',
    ],
    user: [
      'orders:read:own', 'orders:write:own',
      'profile:read:own', 'profile:write:own',
    ],
    readonly: [
      'orders:read:own',
      'profile:read:own',
    ],
  };

  /**
   * Check if user has permission for action on resource.
   * Implements both RBAC and ABAC. 
   */
  async authorize(context: AccessControlContext): Promise<AuthorizationResult> {
    const startTime = Date.now();
    
    try {
      // Step 1: Check explicit deny policies first
      const denyPolicy = this.evaluatePolicies(context, 'deny');
      if (denyPolicy) {
        return this.createDeniedResult(context, denyPolicy, startTime);
      }

      // Step 2: Check role-based permissions (RBAC)
      const rbacResult = this.checkRBAC(context);
      if (! rbacResult.allowed) {
        return this.createDeniedResult(context, 'RBAC', startTime);
      }

      // Step 3: Check attribute-based conditions (ABAC)
      const abacResult = this.checkABAC(context);
      if (! abacResult.allowed) {
        return this.createDeniedResult(context, 'ABAC', startTime);
      }

      // Step 4: Check allow policies
      const allowPolicy = this.evaluatePolicies(context, 'allow');
      if (! allowPolicy && this.requiresExplicitAllow(context)) {
        return this. createDeniedResult(context, 'NO_ALLOW_POLICY', startTime);
      }

      // Step 5: Risk-based access control
      if (context.environment.riskScore && context.environment.riskScore > 0. 8) {
        return this.createDeniedResult(context, 'HIGH_RISK', startTime);
      }

      return this.createAllowedResult(context, startTime);

    } catch (error) {
      // Fail secure - deny on error
      this.logger.error('Authorization error', { context, error });
      return this.createDeniedResult(context, 'ERROR', startTime);
    }
  }

  /**
   * RBAC check - verify user role has required permission.
   */
  private checkRBAC(context: AccessControlContext): { allowed: boolean; reason?: string } {
    const requiredPermission = `${context.resource. type}:${context.action}`;
    
    for (const role of context.roles) {
      const permissions = this.rolePermissions[role] ??  [];
      
      // Check exact match
      if (permissions.includes(requiredPermission)) {
        return { allowed: true };
      }
      
      // Check ownership-scoped permission
      const ownPermission = `${requiredPermission}:own`;
      if (
        permissions.includes(ownPermission) &&
        context. resource.ownerId === context.userId
      ) {
        return { allowed: true };
      }
    }

    return { allowed: false, reason: 'INSUFFICIENT_PERMISSIONS' };
  }

  /**
   * ABAC check - verify attribute-based conditions.
   */
  private checkABAC(context: AccessControlContext): { allowed: boolean; reason?: string } {
    // Example: Time-based access
    const hour = context.environment.timestamp. getHours();
    if (context.resource.type === 'financial_reports' && (hour < 9 || hour > 17)) {
      return { allowed: false, reason: 'OUTSIDE_BUSINESS_HOURS' };
    }

    // Example: Location-based access
    const restrictedIPs = this.config.get('restrictedIPs');
    if (restrictedIPs. includes(context.environment.ipAddress)) {
      return { allowed: false, reason: 'RESTRICTED_LOCATION' };
    }

    // Example: Resource attribute check
    if (
      context.resource. attributes?.classification === 'top-secret' &&
      ! context.attributes. clearanceLevel?. includes('top-secret')
    ) {
      return { allowed: false, reason: 'INSUFFICIENT_CLEARANCE' };
    }

    return { allowed: true };
  }

  /**
   * Log authorization decision for audit. 
   */
  private createAllowedResult(
    context: AccessControlContext,
    startTime: number
  ): AuthorizationResult {
    const result: AuthorizationResult = {
      allowed: true,
      userId: context.userId,
      resource: `${context.resource. type}:${context.resource.id}`,
      action: context.action,
      timestamp: new Date(),
      durationMs: Date. now() - startTime,
    };

    this.auditLogger.info('Authorization granted', result);
    return result;
  }

  private createDeniedResult(
    context: AccessControlContext,
    reason: string,
    startTime: number
  ): AuthorizationResult {
    const result: AuthorizationResult = {
      allowed: false,
      userId: context.userId,
      resource: `${context.resource. type}:${context.resource.id}`,
      action: context.action,
      reason,
      timestamp: new Date(),
      durationMs: Date.now() - startTime,
    };

    this.auditLogger.warn('Authorization denied', result);
    return result;
  }
}
```

### Input Validation & Sanitization

```typescript
// security/input-validator.ts

import { z } from 'zod';
import DOMPurify from 'isomorphic-dompurify';
import sqlstring from 'sqlstring';

/**
 * Comprehensive input validation and sanitization.
 * 
 * Defense in depth: Validate type, format, and sanitize content.
 */
export class InputValidator {
  /**
   * Common validation schemas.
   */
  static readonly schemas = {
    // Email with strict validation
    email: z.string()
      .email('Invalid email format')
      .max(254, 'Email too long')
      . toLowerCase()
      .transform(email => email.trim()),

    // Password with security requirements
    password: z. string()
      . min(12, 'Password must be at least 12 characters')
      .max(128, 'Password too long')
      .regex(/[A-Z]/, 'Password must contain uppercase letter')
      .regex(/[a-z]/, 'Password must contain lowercase letter')
      . regex(/[0-9]/, 'Password must contain number')
      . regex(/[^A-Za-z0-9]/, 'Password must contain special character'),

    // UUID v4
    uuid: z. string()
      . uuid('Invalid ID format'),

    // Safe string (no special characters)
    safeString: z. string()
      . max(1000)
      .regex(/^[a-zA-Z0-9\s\-_. ]+$/, 'Contains invalid characters'),

    // URL with protocol validation
    url: z.string()
      .url('Invalid URL')
      .refine(
        url => url.startsWith('https://'),
        'Only HTTPS URLs allowed'
      ),

    // Phone number (E.164 format)
    phone: z.string()
      .regex(/^\+[1-9]\d{1,14}$/, 'Invalid phone format'),

    // Date in ISO format
    isoDate: z. string()
      . datetime({ message: 'Invalid date format' }),

    // Positive integer
    positiveInt: z.number()
      .int('Must be an integer')
      . positive('Must be positive')
      .max(Number.MAX_SAFE_INTEGER, 'Number too large'),

    // Pagination
    pagination: z.object({
      page: z.number().int().min(1). default(1),
      limit: z.number().int().min(1).max(100).default(20),
      sortBy: z.string().regex(/^[a-zA-Z_]+$/). optional(),
      sortOrder: z.enum(['asc', 'desc']).default('desc'),
    }),
  };

  /**
   * Sanitize HTML content to prevent XSS. 
   */
  static sanitizeHtml(input: string): string {
    return DOMPurify.sanitize(input, {
      ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br'],
      ALLOWED_ATTR: [],
      ALLOW_DATA_ATTR: false,
    });
  }

  /**
   * Sanitize for SQL (use parameterized queries instead when possible).
   */
  static sanitizeSql(input: string): string {
    return sqlstring.escape(input);
  }

  /**
   * Sanitize for shell commands (avoid shell commands when possible).
   */
  static sanitizeShell(input: string): string {
    // Whitelist approach - only allow safe characters
    return input. replace(/[^a-zA-Z0-9\-_. ]/g, '');
  }

  /**
   * Sanitize for logging (remove/mask sensitive data). 
   */
  static sanitizeForLogging(obj: Record<string, unknown>): Record<string, unknown> {
    const sensitiveFields = [
      'password', 'token', 'secret', 'apiKey', 'authorization',
      'creditCard', 'ssn', 'pin', 'cvv',
    ];
    
    const sanitized = { ...obj };
    
    for (const key of Object.keys(sanitized)) {
      const lowerKey = key. toLowerCase();
      
      if (sensitiveFields.some(field => lowerKey.includes(field))) {
        sanitized[key] = '[REDACTED]';
      } else if (typeof sanitized[key] === 'object' && sanitized[key] !== null) {
        sanitized[key] = this.sanitizeForLogging(sanitized[key] as Record<string, unknown>);
      }
    }
    
    return sanitized;
  }

  /**
   * Validate and sanitize user input with detailed errors.
   */
  static validate<T>(
    schema: z.ZodSchema<T>,
    input: unknown,
    options: { stripUnknown?: boolean } = {}
  ): T {
    try {
      return schema.parse(input);
    } catch (error) {
      if (error instanceof z. ZodError) {
        const formattedErrors = error.errors.map(err => ({
          field: err.path. join('.'),
          message: err.message,
        }));
        
        throw new ValidationError('Validation failed', formattedErrors);
      }
      throw error;
    }
  }

  /**
   * Detect potential injection attacks.
   */
  static detectInjection(input: string): { detected: boolean; type?: string } {
    const patterns = {
      sql: /('|"|;|--|\bOR\b|\bAND\b|\bUNION\b|\bSELECT\b|\bDROP\b)/i,
      xss: /(<script|javascript:|on\w+\s*=)/i,
      command: /(\||;|`|\$\(|&&)/,
      path: /(\.\.\/|\.\.\\)/,
      ldap: /([)(|*\\])/,
    };

    for (const [type, pattern] of Object.entries(patterns)) {
      if (pattern.test(input)) {
        return { detected: true, type };
      }
    }

    return { detected: false };
  }
}

// Usage example with request validation
const createUserSchema = z.object({
  email: InputValidator.schemas.email,
  password: InputValidator. schemas.password,
  name: z.string()
    .min(1, 'Name is required')
    . max(100, 'Name too long')
    .transform(name => InputValidator.sanitizeHtml(name. trim())),
  bio: z.string()
    .max(500)
    .optional()
    .transform(bio => bio ?  InputValidator.sanitizeHtml(bio) : undefined),
});

export type CreateUserInput = z.infer<typeof createUserSchema>;
```

---

## 🔒 SECURITY SCANNING

### SAST Configuration (Semgrep)

```yaml
# scanning/sast/semgrep-rules/custom-rules.yaml

rules:
  # SQL Injection Detection
  - id: sql-injection-risk
    patterns:
      - pattern-either:
          - pattern: $QUERY = "..." + $INPUT + "..."
          - pattern: $QUERY = f"... {$INPUT}..."
          - pattern: $QUERY = `...${$INPUT}...`
    message: "Potential SQL injection vulnerability.  Use parameterized queries."
    languages: [python, javascript, typescript]
    severity: ERROR
    metadata:
      cwe: "CWE-89"
      owasp: "A03:2021 - Injection"

  # Hardcoded Secrets
  - id: hardcoded-secret
    patterns:
      - pattern-either:
          - pattern: $VAR = "AKIA..."  # AWS Access Key
          - pattern: $VAR = "sk_live_..."  # Stripe Key
          - pattern: password = "..."
          - pattern: api_key = "..."
          - pattern: secret = "..."
    message: "Potential hardcoded secret detected. Use environment variables or secret manager."
    languages: [python, javascript, typescript, java, go]
    severity: ERROR
    metadata:
      cwe: "CWE-798"
      owasp: "A07:2021 - Identification and Authentication Failures"

  # Insecure Crypto
  - id: weak-crypto
    patterns:
      - pattern-either:
          - pattern: crypto. createHash('md5')
          - pattern: crypto.createHash('sha1')
          - pattern: hashlib.md5(...)
          - pattern: hashlib.sha1(...)
    message: "Weak cryptographic algorithm.  Use SHA-256 or stronger."
    languages: [python, javascript, typescript]
    severity: WARNING
    metadata:
      cwe: "CWE-328"

  # Missing Authentication
  - id: missing-auth-check
    patterns:
      - pattern: |
          @app.route(...)
          def $FUNC(...):
            ... 
      - pattern-not: |
          @app.route(...)
          @login_required
          def $FUNC(...):
            ...
    message: "Endpoint may be missing authentication. Add @login_required decorator."
    languages: [python]
    severity: WARNING
    metadata:
      cwe: "CWE-306"

  # Sensitive Data Exposure
  - id: sensitive-data-log
    patterns:
      - pattern-either:
          - pattern: console.log(... , password, ...)
          - pattern: console.log(... , $FUNC(...password.. .), ...)
          - pattern: logger.info(..., password=.. ., ...)
          - pattern: print(...password...)
    message: "Potential sensitive data in logs.  Sanitize before logging."
    languages: [python, javascript, typescript]
    severity: ERROR
    metadata:
      cwe: "CWE-532"
```

### Dependency Scanning (GitHub Actions)

```yaml
# . github/workflows/security-scan.yml

name: Security Scanning

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 6 * * *'  # Daily at 6 AM UTC

jobs:
  # Static Application Security Testing
  sast:
    name: SAST Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Semgrep
        uses: returntocorp/semgrep-action@v1
        with:
          config: >-
            p/security-audit
            p/secrets
            p/owasp-top-ten
            ./scanning/sast/semgrep-rules/
        env:
          SEMGREP_APP_TOKEN: ${{ secrets.SEMGREP_APP_TOKEN }}

      - name: Run CodeQL
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript, typescript

  # Software Composition Analysis
  sca:
    name: Dependency Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Snyk
        uses: snyk/actions/node@master
        continue-on-error: true
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high --json-file-output=snyk-results.json

      - name: Run npm audit
        run: npm audit --audit-level=high

      - name: Run Trivy
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: 'fs'
          scan-ref: '.'
          severity: 'CRITICAL,HIGH'
          format: 'sarif'
          output: 'trivy-results.sarif'

      - name: Upload Trivy results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: 'trivy-results.sarif'

  # Container Scanning
  container-scan:
    name: Container Scan
    runs-on: ubuntu-latest
    if: github.event_name == 'push'
    steps:
      - uses: actions/checkout@v4

      - name: Build image
        run: docker build -t app:${{ github.sha }} .

      - name: Run Trivy container scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'app:${{ github.sha }}'
          format: 'sarif'
          output: 'trivy-container. sarif'
          severity: 'CRITICAL,HIGH'

      - name: Run Grype
        uses: anchore/scan-action@v3
        with:
          image: 'app:${{ github.sha }}'
          severity-cutoff: high

  # Infrastructure as Code Scanning
  iac-scan:
    name: IaC Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Run Checkov
        uses: bridgecrewio/checkov-action@master
        with:
          directory: infrastructure/terraform
          framework: terraform
          output_format: sarif
          output_file_path: checkov-results.sarif

      - name: Run tfsec
        uses: aquasecurity/tfsec-action@v1. 0.0
        with:
          working-directory: infrastructure/terraform
          soft-fail: false

  # Secret Scanning
  secret-scan:
    name: Secret Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        uses: gitleaks/gitleaks-action@v2
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

      - name: Run TruffleHog
        uses: trufflesecurity/trufflehog@main
        with:
          path: ./
          base: ${{ github.event.repository.default_branch }}
          head: HEAD
          extra_args: --only-verified

  # DAST (on staging)
  dast:
    name: DAST Scan
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: staging
    steps:
      - name: Run OWASP ZAP
        uses: zaproxy/action-full-scan@v0.7.0
        with:
          target: ${{ secrets.STAGING_URL }}
          rules_file_name: '. zap/rules.tsv'
          cmd_options: '-a'

  # Aggregate and Report
  report:
    name: Security Report
    needs: [sast, sca, container-scan, iac-scan, secret-scan]
    runs-on: ubuntu-latest
    if: always()
    steps:
      - name: Aggregate Results
        run: |
          echo "## Security Scan Summary" >> $GITHUB_STEP_SUMMARY
          echo "| Scan Type | Status |" >> $GITHUB_STEP_SUMMARY
          echo "|-----------|--------|" >> $GITHUB_STEP_SUMMARY
          echo "| SAST | ${{ needs.sast.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| SCA | ${{ needs.sca.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Container | ${{ needs.container-scan.result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| IaC | ${{ needs. iac-scan. result }} |" >> $GITHUB_STEP_SUMMARY
          echo "| Secrets | ${{ needs.secret-scan. result }} |" >> $GITHUB_STEP_SUMMARY

      - name: Notify on Failure
        if: failure()
        uses: slackapi/slack-github-action@v1
        with:
          payload: |
            {
              "text": "🚨 Security scan failed! Check GitHub Actions for details.",
              "channel": "#security-alerts"
            }
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SECURITY_SLACK_WEBHOOK }}
```

---

## 🚨 INCIDENT RESPONSE

### Incident Response Runbook

```markdown
# Incident Response Runbook

## Severity Classification

| Severity | Description | Response Time | Examples |
|----------|-------------|---------------|----------|
| P1 - Critical | Active breach, data loss, complete outage | 15 min | Ransomware, active attacker, data exfiltration |
| P2 - High | Significant security issue, partial outage | 1 hour | Vulnerability exploitation, credential compromise |
| P3 - Medium | Limited impact security issue | 4 hours | Suspicious activity, failed attacks |
| P4 - Low | Minor security issue | 24 hours | Policy violation, minor misconfiguration |

---

## Incident Response Phases

### Phase 1: Detection & Triage (15-30 min)

#### Initial Assessment
```
□ Confirm the incident is real (not false positive)
□ Classify severity level
□ Identify affected systems/data
□ Determine if incident is ongoing
□ Notify incident commander
```

#### Key Questions
- What happened?
- When did it start? 
- What systems are affected?
- Is customer data at risk? 
- Is the attacker still active?

#### Communication
```
□ Alert on-call security engineer
□ Create incident channel (#incident-YYYYMMDD-brief)
□ Notify stakeholders per severity matrix
□ Begin incident log
```

---

### Phase 2: Containment (30-60 min)

#### Immediate Actions
```
□ Isolate affected systems (if active threat)
□ Revoke compromised credentials
□ Block malicious IPs/domains
□ Preserve evidence (logs, memory dumps)
□ Enable enhanced logging
```

#### Containment Strategies

**For Compromised Server:**
```bash
# Isolate network (keep accessible for investigation)
iptables -I INPUT -j DROP
iptables -I OUTPUT -j DROP
iptables -I INPUT -s <your-ip> -j ACCEPT
iptables -I OUTPUT -d <your-ip> -j ACCEPT

# Capture memory dump
sudo dd if=/dev/mem of=/evidence/memory.dump bs=1M

# Capture disk image
sudo dd if=/dev/sda of=/evidence/disk. img bs=4M
```

**For Credential Compromise:**
```bash
# Revoke all sessions for user
aws iam delete-login-profile --user-name <username>
aws iam list-access-keys --user-name <username>
aws iam delete-access-key --user-name <username> --access-key-id <key-id>

# Force password reset
./scripts/force-password-reset.sh <user-id>
```

---

### Phase 3: Eradication (1-4 hours)

#### Root Cause Analysis
```
□ Identify attack vector
□ Identify all affected systems
□ Determine scope of compromise
□ Identify persistence mechanisms
□ Document findings
```

#### Remediation
```
□ Remove malware/backdoors
□ Patch vulnerabilities
□ Rotate all potentially compromised credentials
□ Update security controls
□ Verify clean state
```

---

### Phase 4: Recovery (2-8 hours)

#### System Restoration
```
□ Restore from clean backups (if needed)
□ Verify system integrity
□ Re-enable services gradually
□ Monitor for reinfection
□ Validate functionality
```

#### Verification Checklist
```
□ All IOCs removed
□ Vulnerabilities patched
□ Credentials rotated
□ Monitoring enhanced
□ No ongoing suspicious activity
```

---

### Phase 5: Post-Incident (24-72 hours)

#### Documentation
```
□ Complete incident timeline
□ Document root cause
□ Document all actions taken
□ Calculate business impact
□ Identify lessons learned
```

#### Post-Mortem Template
```markdown
## Incident Post-Mortem: [INCIDENT-ID]

### Summary
- **Date:** 
- **Duration:** 
- **Severity:** 
- **Impact:** 

### Timeline
| Time | Event |
|------|-------|
| 00:00 | Initial detection |
| 00:15 | Incident confirmed |
| ...  | ... |

### Root Cause
[Detailed description of root cause]

### Impact
- Systems affected:
- Data affected:
- Users affected:
- Financial impact:

### Response Evaluation
- What went well:
- What could improve:

### Action Items
| Action | Owner | Due Date | Status |
|--------|-------|----------|--------|
| | | | |

### Lessons Learned
1. 
2.
3.
```
```

---

## 📊 SECURITY MONITORING

### SIEM Rules (Splunk/Elastic)

```yaml
# monitoring/siem-rules/detection-rules.yaml

rules:
  # Brute Force Detection
  - name: Brute Force Login Attempt
    description: Multiple failed login attempts from single IP
    query: |
      source="auth_logs" action="login_failed"
      | stats count by src_ip, user
      | where count > 10
    severity: high
    threshold: 10
    window: 5m
    actions:
      - alert: security-team
      - block_ip: temporary

  # Privilege Escalation
  - name: Unusual Privilege Escalation
    description: User granted admin privileges outside normal process
    query: |
      source="audit_logs" action="role_change" new_role="admin"
      | where NOT match(actor, "^system-")
      | where NOT match(ticket_id, "^APPROVED-")
    severity: critical
    actions:
      - alert: security-team
      - alert: management
      - suspend_user: pending_review

  # Data Exfiltration
  - name: Large Data Export
    description: Unusually large data export by single user
    query: |
      source="app_logs" action="export" OR action="download"
      | stats sum(bytes) as total_bytes by user
      | where total_bytes > 100000000
    severity: high
    threshold: 100MB
    window: 1h
    actions:
      - alert: security-team
      - rate_limit: user

  # Suspicious API Access
  - name: API Access Anomaly
    description: Unusual API access pattern
    query: |
      source="api_logs"
      | stats count by user, endpoint
      | anomalydetection count
      | where anomaly_score > 3
    severity: medium
    actions:
      - alert: security-team

  # After Hours Access
  - name: Sensitive Data Access After Hours
    description: Access to sensitive data outside business hours
    query: |
      source="data_access_logs" classification="confidential" OR classification="restricted"
      | where date_hour < 7 OR date_hour > 19
      | where date_wday=0 OR date_wday=6
    severity: medium
    actions:
      - alert: security-team

  # Impossible Travel
  - name: Impossible Travel Detection
    description: User login from geographically impossible locations
    query: |
      source="auth_logs" action="login_success"
      | iplocation src_ip
      | streamstats current=f last(City) as prev_city, last(_time) as prev_time by user
      | eval distance = haversine(lat, lon, prev_lat, prev_lon)
      | eval time_diff = _time - prev_time
      | eval speed = distance / (time_diff / 3600)
      | where speed > 1000
    severity: high
    actions:
      - alert: security-team
      - require_mfa: next_action
```

---

## 🧪 SECURITY VERIFICATION PLAN

```markdown
## 🧪 Security Verification

### Security Testing Checklist

#### Authentication Testing
- [ ] Password policy enforcement
- [ ] Account lockout mechanism
- [ ] Session management security
- [ ] MFA implementation
- [ ] Password reset flow security

#### Authorization Testing
- [ ] Role-based access control
- [ ] Privilege escalation prevention
- [ ] IDOR vulnerability testing
- [ ] API authorization

#### Input Validation Testing
- [ ] SQL injection
- [ ] XSS (reflected, stored, DOM)
- [ ] Command injection
- [ ] Path traversal
- [ ] XXE injection

#### Cryptography Testing
- [ ] TLS configuration
- [ ] Certificate validation
- [ ] Encryption at rest
- [ ] Key management

#### Security Configuration
- [ ] Security headers
- [ ] CORS configuration
- [ ] Error handling
- [ ] Logging configuration

### Penetration Test Scope
- [ ] External network
- [ ] Internal network
- [ ] Web application
- [ ] API endpoints
- [ ] Mobile application
- [ ] Cloud infrastructure
```

---

## 🚫 SECURITY ANTI-PATTERNS

| Avoid | Why | Instead |
|-------|-----|---------|
| Security through obscurity | Doesn't work | Defense in depth |
| Rolling your own crypto | Will have flaws | Use established libraries |
| Hardcoded secrets | Easily discovered | Secret management |
| Trust user input | Injection attacks | Validate everything |
| Generic error messages | May hide issues | Log details, show generic |
| Single layer of security | Single point of failure | Defense in depth |
| Infrequent patching | Known vulnerabilities | Regular patch cycles |
| No security logging | Blind to attacks | Comprehensive audit logs |
| Static credentials | Credential stuffing | Rotate regularly |
| Overly permissive access | Least privilege violation | Minimal permissions |