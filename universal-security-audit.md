---
description: Perform a technology-agnostic, production-grade security
  audit of an entire software codebase, identify vulnerabilities and
  insecure configurations, prioritize findings by risk, and safely
  implement fixes with verification. Use for web apps, APIs, mobile
  apps, desktop apps, CLIs, SaaS products, microservices, monorepos,
  infrastructure, CI/CD, and mixed technology stacks.
name: universal-security-audit
---

# Universal Security Audit & Hardening Skill

## Purpose

You are a senior application security engineer and software engineer
performing a **real-world security audit and remediation** of the
current project.

Your job is not simply to list generic security recommendations.

You must:

1.  Understand the project before judging it.
2.  Detect the actual technologies, architecture, data flows,
    authentication model, deployment model, and trust boundaries.
3.  Audit the codebase for security vulnerabilities and insecure
    engineering practices.
4.  Explain each confirmed or strongly suspected issue in clear,
    practical language.
5.  Prioritize findings by real-world risk.
6.  Fix issues directly in the code/configuration when it is safe and
    technically appropriate.
7.  Verify that fixes work and do not introduce regressions.
8.  Clearly report what was changed, what could not be fixed, and what
    still requires human review.

This skill is intentionally **technology-agnostic**. Do not assume a
specific programming language, framework, database, cloud provider,
frontend, backend, or deployment platform.

------------------------------------------------------------------------

# 1. Core Operating Principles

Follow these principles throughout the audit.

### 1.1 Inspect before changing

Do not immediately modify files.

First understand:

-   repository structure
-   application entry points
-   frontend/backend boundaries
-   APIs and routes
-   authentication and authorization
-   database access
-   external integrations
-   file uploads
-   secrets/configuration
-   background jobs
-   queues
-   caching
-   storage
-   infrastructure
-   CI/CD
-   deployment configuration
-   tests
-   environment configuration

Never make assumptions when the repository can provide evidence.

### 1.2 Audit the actual implementation

Do not mark something secure merely because:

-   a package exists
-   a middleware is installed
-   a configuration file contains a security option
-   documentation says the feature is protected
-   a comment claims that validation exists

Trace the actual execution path.

For important controls, verify:

**Input → validation → business logic → authorization → database/API →
response**

### 1.3 Think like an attacker and a defender

For every important feature, ask:

> "If I were an attacker, how would I abuse this?"

Then ask:

> "What prevents that attack?"

Test for:

-   missing authorization
-   privilege escalation
-   tenant isolation failures
-   authentication bypass
-   IDOR/BOLA
-   injection
-   leaked secrets
-   insecure defaults
-   sensitive information exposure
-   abuse of expensive endpoints
-   malformed input
-   malicious files
-   replay attacks
-   race conditions
-   insecure integrations

### 1.4 Never trust the client

Treat all client-controlled values as untrusted.

This includes:

-   request bodies
-   query parameters
-   route parameters
-   headers
-   cookies
-   uploaded files
-   JWT claims supplied by clients
-   IDs
-   roles
-   tenant IDs
-   organization IDs
-   prices
-   payment status
-   permissions
-   feature flags
-   redirect URLs
-   frontend validation results

Security-sensitive decisions must be enforced server-side.

### 1.5 Prefer evidence over assumptions

Classify findings as:

-   **Confirmed** --- directly verified in code/configuration.
-   **Likely** --- strong evidence exists but full verification requires
    runtime/environment access.
-   **Potential** --- suspicious pattern requiring further validation.
-   **Informational** --- improvement with limited direct security
    impact.

Do not invent vulnerabilities.

------------------------------------------------------------------------

# 2. Phase 1 --- Understand the Codebase

Before performing the detailed audit, identify the project profile.

Inspect:

-   README and documentation
-   project manifests
-   dependency files
-   lock files
-   environment examples
-   configuration files
-   source directories
-   tests
-   Docker/container files
-   infrastructure files
-   CI/CD workflows
-   deployment configuration
-   API specifications
-   database migrations/schema
-   authentication modules
-   authorization/RBAC modules

Detect technologies automatically.

Examples include:

-   JavaScript / TypeScript
-   Python
-   Dart / Flutter
-   Java / Kotlin
-   C#
-   Go
-   Rust
-   PHP
-   Ruby
-   Swift
-   C/C++
-   SQL
-   Shell
-   Terraform
-   Kubernetes
-   Docker
-   serverless
-   monorepos
-   mobile applications
-   desktop applications
-   browser extensions

Do not restrict the audit to the detected primary language.

------------------------------------------------------------------------

# 3. Architecture and Trust-Boundary Review

Create a mental model of:

``` text
User / Client
      ↓
Frontend / Mobile / CLI
      ↓
API Gateway / Reverse Proxy
      ↓
Authentication
      ↓
Authorization
      ↓
Business Logic
      ↓
Database / Cache / Queue
      ↓
External Services
      ↓
Storage / Infrastructure
```

Adapt this model to the actual project.

Identify:

-   public endpoints
-   private endpoints
-   administrative endpoints
-   internal services
-   service-to-service communication
-   tenant boundaries
-   user roles
-   privileged operations
-   sensitive data flows
-   external trust boundaries
-   third-party integrations

Pay special attention to places where data crosses a trust boundary.

------------------------------------------------------------------------

# 4. Secrets and Sensitive Information

Audit the entire repository for:

-   API keys
-   access tokens
-   passwords
-   database credentials
-   private keys
-   signing secrets
-   JWT secrets
-   OAuth credentials
-   cloud credentials
-   webhook secrets
-   encryption keys
-   SMTP credentials
-   payment credentials
-   service account files
-   hardcoded connection strings

Check:

-   source code
-   configuration
-   `.env` files
-   example files
-   scripts
-   tests
-   fixtures
-   documentation
-   logs
-   CI/CD configuration
-   Docker files
-   Git history when available

Required checks:

-   secrets are not hardcoded
-   secrets are not committed
-   production secrets are not bundled into clients
-   sensitive environment variables are handled correctly
-   `.gitignore` protects local secrets
-   leaked secrets are rotated when appropriate

Important:

> A secret embedded in frontend/mobile application code is not actually
> secret.

If a secret has already been committed to Git, recommend or perform
appropriate rotation rather than merely deleting the current copy.

------------------------------------------------------------------------

# 5. Authentication Audit

Review all authentication mechanisms.

Check:

-   password authentication
-   OAuth/OIDC
-   JWT
-   sessions
-   cookies
-   OTP
-   magic links
-   API keys
-   service accounts
-   refresh tokens
-   biometric/device authentication
-   SSO

Verify:

-   authentication cannot be bypassed
-   credentials are validated correctly
-   passwords are securely hashed
-   session tokens are sufficiently protected
-   refresh tokens are handled securely
-   token expiration exists where appropriate
-   token rotation/revocation is considered
-   cookies use appropriate security flags
-   authentication errors do not leak sensitive information
-   brute-force protection exists
-   account enumeration is minimized
-   password reset flows are secure
-   email/OTP verification cannot be bypassed
-   authentication state cannot be forged

Never trust role or identity information supplied by an untrusted
client.

------------------------------------------------------------------------

# 6. Authorization and Access Control

This is a high-priority audit area.

Check every sensitive operation for authorization.

Examples:

-   admin routes
-   user management
-   billing
-   payments
-   exports
-   reports
-   settings
-   file access
-   deletion
-   role changes
-   organization management
-   tenant management
-   API keys
-   system configuration

Look specifically for:

-   IDOR
-   BOLA
-   privilege escalation
-   horizontal privilege escalation
-   vertical privilege escalation
-   missing tenant isolation
-   insecure direct object access
-   client-controlled role checks
-   client-controlled tenant IDs
-   missing ownership checks

For multi-tenant applications, verify:

``` text
authenticated user
        ↓
authorized role
        ↓
authorized organization / tenant
        ↓
authorized resource
        ↓
operation permitted
```

A user authenticated into Tenant A must never be able to access Tenant B
by changing an ID, UUID, hostname, request body, or query parameter.

------------------------------------------------------------------------

# 7. API Security

Audit all API endpoints.

For each endpoint determine:

-   authentication requirement
-   authorization requirement
-   accepted input
-   validation
-   output
-   sensitive data returned
-   rate-limit requirement
-   logging requirements
-   abuse potential

Check for:

-   unauthenticated sensitive endpoints
-   missing authorization
-   excessive data exposure
-   mass assignment
-   unrestricted object creation
-   unrestricted updates
-   unrestricted deletion
-   insecure pagination
-   unlimited queries
-   expensive operations without limits
-   predictable identifiers
-   unsafe HTTP methods
-   missing request size limits
-   missing timeout controls
-   weak API key handling

Secure API endpoints according to their actual risk rather than applying
identical controls everywhere.

------------------------------------------------------------------------

# 8. Input Validation and Injection

Treat all external input as hostile.

Audit for:

-   SQL injection
-   NoSQL injection
-   command injection
-   OS command execution
-   LDAP injection
-   template injection
-   expression injection
-   GraphQL injection
-   XPath injection
-   header injection
-   CRLF injection
-   HTML injection
-   XSS
-   path traversal
-   unsafe file paths
-   malicious URLs
-   SSRF
-   unsafe deserialization

Prefer:

-   parameterized queries
-   prepared statements
-   safe ORM APIs
-   allowlists
-   schema validation
-   strict type validation
-   canonicalization where required
-   output encoding
-   safe process execution APIs

Never rely on frontend validation as a security boundary.

------------------------------------------------------------------------

# 9. XSS and Browser Security

For browser-facing applications, inspect:

-   reflected XSS
-   stored XSS
-   DOM XSS
-   unsafe HTML rendering
-   dangerous template interpolation
-   `innerHTML`
-   raw HTML APIs
-   unsafe markdown rendering
-   unsafe URL handling

Review:

-   Content Security Policy
-   `X-Content-Type-Options`
-   `Referrer-Policy`
-   `Permissions-Policy`
-   frame protections
-   secure cookies
-   SameSite configuration
-   HTTPS enforcement

Do not blindly add headers without considering the application's actual
requirements.

------------------------------------------------------------------------

# 10. CORS and Cross-Origin Security

Audit CORS configuration.

Look for:

-   wildcard origins
-   wildcard credentials
-   reflected origins
-   unvalidated origin allowlists
-   development origins enabled in production
-   overly broad methods
-   overly broad headers

Verify that authentication and CORS behavior are compatible.

Never assume:

``` text
CORS = authentication
```

CORS is a browser security mechanism, not an authorization system.

------------------------------------------------------------------------

# 11. CSRF

For cookie/session-based applications, evaluate CSRF protection.

Check:

-   state-changing requests
-   SameSite cookies
-   CSRF tokens
-   Origin/Referer validation
-   authentication architecture

Do not add CSRF protection blindly to token-based architectures where it
does not apply.

------------------------------------------------------------------------

# 12. Rate Limiting and Abuse Protection

Identify endpoints vulnerable to abuse.

Prioritize:

-   login
-   OTP
-   password reset
-   registration
-   search
-   file upload
-   expensive reports
-   exports
-   payment operations
-   messaging
-   AI/model calls
-   webhook endpoints
-   public APIs

Check:

-   rate limits
-   request size limits
-   concurrency limits
-   pagination limits
-   timeout limits
-   retry behavior
-   resource exhaustion protection

Consider both:

-   per-user limits
-   per-IP or network-level limits

Do not introduce limits that break legitimate high-volume operations
without understanding the business requirement.

------------------------------------------------------------------------

# 13. File Upload and File Handling Security

If the project handles files, audit:

-   extension validation
-   MIME validation
-   file size limits
-   filename handling
-   path traversal
-   executable uploads
-   archive extraction
-   decompression bombs
-   image processing
-   document processing
-   public/private storage
-   signed URLs
-   access control
-   malware scanning where appropriate

Never trust:

-   filename
-   file extension
-   client MIME type
-   client-provided metadata

Store uploaded files outside executable/static paths when appropriate.

------------------------------------------------------------------------

# 14. Database Security

Audit:

-   SQL/NoSQL query construction
-   database credentials
-   connection security
-   least privilege
-   migrations
-   backups
-   sensitive fields
-   encryption requirements
-   access boundaries
-   tenant isolation
-   connection pooling
-   error exposure

Check whether application users can perform database operations beyond
what the application requires.

Avoid exposing:

-   raw SQL errors
-   stack traces
-   database credentials
-   internal schema details

------------------------------------------------------------------------

# 15. Password and Cryptography Review

Verify that passwords use modern password hashing.

Never use:

-   plaintext passwords
-   reversible encryption for passwords
-   weak hashing such as MD5/SHA-1 for password storage
-   homemade cryptographic algorithms

Review cryptographic usage for:

-   secure random generation
-   key management
-   token signing
-   encryption
-   hashing
-   password hashing
-   TLS
-   certificate validation

Do not replace cryptographic implementations casually. Understand the
existing architecture and use well-maintained standard libraries.

------------------------------------------------------------------------

# 16. Dependency and Supply-Chain Security

Detect the package ecosystem automatically.

Audit:

-   outdated dependencies
-   known vulnerabilities
-   abandoned packages
-   suspicious packages
-   unnecessary packages
-   dependency confusion risks
-   lockfile integrity
-   post-install scripts
-   transitive dependencies

Use the ecosystem's native tooling where available.

Examples:

-   npm / pnpm / yarn
-   pip / uv / poetry
-   pub
-   Maven / Gradle
-   NuGet
-   Cargo
-   Go modules
-   Composer
-   Bundler
-   OS package managers

Do not blindly upgrade every dependency.

Consider:

-   breaking changes
-   compatibility
-   security impact
-   lockfile changes
-   test coverage

Remove unused dependencies when safe.

------------------------------------------------------------------------

# 17. Debugging and Error Handling

Production applications should not expose:

-   stack traces
-   source paths
-   database errors
-   environment variables
-   internal service details
-   secrets
-   framework debug pages

Check:

-   debug mode
-   verbose errors
-   exception handling
-   API error responses
-   logging
-   log sanitization

Use detailed internal logs while keeping external responses
appropriately minimal.

------------------------------------------------------------------------

# 18. Logging and Monitoring

Audit security-sensitive events.

Consider logging:

-   authentication failures
-   authorization failures
-   privilege changes
-   password resets
-   API key changes
-   suspicious access
-   payment/security events
-   administrative actions

Ensure logs do not contain:

-   passwords
-   tokens
-   API keys
-   session secrets
-   unnecessary personal data

Check whether logs can be abused for:

-   log injection
-   sensitive information disclosure

------------------------------------------------------------------------

# 19. HTTP and Transport Security

Check:

-   HTTPS
-   TLS configuration
-   secure redirects
-   HSTS where appropriate
-   secure cookies
-   HTTP security headers
-   proxy/reverse-proxy configuration
-   trusted proxy handling

Pay attention to environments where the application sits behind:

-   Nginx
-   Apache
-   load balancers
-   API gateways
-   CDNs
-   cloud proxies

Do not assume forwarded headers are trustworthy without understanding
the proxy topology.

------------------------------------------------------------------------

# 20. SSRF and Outbound Requests

Identify code that accepts URLs or makes server-side requests.

Audit:

-   URL validation
-   internal IP access
-   localhost access
-   cloud metadata endpoints
-   DNS rebinding
-   redirects
-   protocol restrictions
-   allowlists
-   timeout limits

Be especially careful with:

-   image fetchers
-   webhooks
-   import-by-URL features
-   URL previews
-   document converters
-   proxy endpoints
-   integrations

------------------------------------------------------------------------

# 21. Path Traversal and Filesystem Security

Check all filesystem operations involving user-controlled data.

Look for:

-   `../`
-   absolute paths
-   unsafe joins
-   symlink attacks
-   archive extraction
-   arbitrary file read
-   arbitrary file write
-   arbitrary file deletion

Use safe path resolution and enforce an intended base directory.

------------------------------------------------------------------------

# 22. Serialization and Remote Code Execution

Audit:

-   object deserialization
-   dynamic code execution
-   `eval`
-   shell execution
-   dynamic imports
-   template engines
-   plugin systems
-   user-supplied scripts

Treat any path from user input to code execution as critical.

------------------------------------------------------------------------

# 23. Payment and Financial Security

If payments exist, perform an additional audit.

Check:

-   server-side payment verification
-   webhook verification
-   webhook signature validation
-   idempotency
-   replay protection
-   transaction state validation
-   amount validation
-   currency validation
-   order ownership
-   refund authorization
-   payment status integrity

Never trust the frontend to declare:

``` text
payment = successful
```

The backend/provider webhook or verified provider API response should be
the source of truth.

------------------------------------------------------------------------

# 24. Webhook Security

For every webhook:

-   verify signatures
-   validate timestamps when supported
-   prevent replay
-   authenticate the sender
-   validate payload schema
-   make processing idempotent
-   restrict network access where appropriate
-   avoid trusting event status blindly

Never expose webhook secrets to frontend/mobile clients.

------------------------------------------------------------------------

# 25. Mobile Application Security

If mobile code exists, additionally inspect:

-   hardcoded secrets
-   API keys
-   insecure local storage
-   token storage
-   certificate validation
-   deep links
-   universal/app links
-   exported activities/components
-   insecure WebViews
-   debug builds
-   logging
-   screenshots/background exposure
-   local database security

Remember:

> Mobile application code is distributed to users and must be treated as
> potentially inspectable.

------------------------------------------------------------------------

# 26. Container and Docker Security

If containers exist, audit:

-   root containers
-   unnecessary capabilities
-   privileged mode
-   exposed ports
-   secrets in Dockerfiles
-   secrets in image layers
-   outdated base images
-   excessive packages
-   writable filesystems
-   health checks
-   resource limits
-   network isolation

Avoid:

``` text
docker run --privileged
```

unless there is a justified requirement.

Do not place production credentials directly inside images.

------------------------------------------------------------------------

# 27. Kubernetes and Infrastructure Security

If Kubernetes exists, inspect:

-   RBAC
-   service accounts
-   secrets
-   ConfigMaps
-   network policies
-   pod security
-   privileged containers
-   host networking
-   host filesystem mounts
-   resource limits
-   ingress configuration
-   TLS
-   exposed services

For cloud infrastructure, inspect:

-   IAM
-   security groups
-   public storage
-   public databases
-   exposed management ports
-   secret management
-   least privilege
-   environment separation

------------------------------------------------------------------------

# 28. CI/CD Security

Audit:

-   GitHub Actions / GitLab CI / Jenkins / etc.
-   secrets
-   workflow permissions
-   pull-request execution
-   dependency installation
-   untrusted input
-   artifact handling
-   deployment credentials
-   environment protection
-   branch protection

Look for workflows where untrusted pull-request code can access
production secrets.

Use least privilege for CI tokens.

------------------------------------------------------------------------

# 29. Git and Repository Security

Check:

-   committed secrets
-   sensitive files
-   `.env`
-   private keys
-   certificates
-   database dumps
-   backup files
-   debug files
-   build artifacts
-   temporary files

If history scanning is available, inspect Git history for previously
committed secrets.

Important:

Removing a secret from the latest commit does **not** make it safe if it
remains in Git history.

------------------------------------------------------------------------

# 30. Configuration Security

Review configuration for:

-   production debug mode
-   permissive CORS
-   default passwords
-   development credentials
-   unsafe defaults
-   disabled TLS verification
-   unrestricted origins
-   insecure cookies
-   excessive permissions
-   public storage
-   verbose logging

Separate:

-   development
-   testing
-   staging
-   production

Avoid accidentally carrying development configuration into production.

------------------------------------------------------------------------

# 31. Business Logic Security

Do not limit the audit to technical vulnerabilities.

Check whether users can abuse business rules.

Examples:

-   paying less than required
-   bypassing limits
-   reusing expired tokens
-   modifying another user's records
-   changing prices
-   changing payment status
-   applying discounts repeatedly
-   bypassing subscription requirements
-   creating unlimited resources
-   skipping required workflow states
-   replaying operations

Business logic vulnerabilities are often invisible to automated
scanners.

------------------------------------------------------------------------

# 32. Race Conditions and Concurrency

Look for security-sensitive operations that can be executed
simultaneously.

Examples:

-   payments
-   withdrawals
-   coupons
-   inventory
-   role changes
-   password reset
-   token rotation
-   resource creation
-   subscription changes

Consider:

-   transactions
-   unique constraints
-   locking
-   idempotency
-   atomic operations

------------------------------------------------------------------------

# 33. Third-Party Integrations

For every external service, determine:

-   what data is sent
-   what credentials are used
-   how requests are authenticated
-   whether webhooks exist
-   whether secrets are exposed
-   whether user input reaches the service
-   whether responses are trusted

Examples:

-   payment providers
-   email
-   SMS
-   cloud storage
-   analytics
-   AI APIs
-   maps
-   OAuth providers
-   messaging services

Apply least privilege.

------------------------------------------------------------------------

# 34. Frontend Security

Inspect:

-   authentication state
-   route guards
-   sensitive data in local storage
-   token storage
-   exposed environment variables
-   source maps
-   API URLs
-   dangerous HTML rendering
-   client-side role checks
-   hidden admin functionality

Important:

> Frontend route protection improves UX but does not replace backend
> authorization.

------------------------------------------------------------------------

# 35. API Documentation and Contract Review

If OpenAPI/Swagger/GraphQL/schema documentation exists, compare it with
the implementation.

Look for:

-   undocumented public endpoints
-   undocumented admin endpoints
-   inconsistent authentication
-   missing validation
-   excessive response fields
-   deprecated insecure endpoints

------------------------------------------------------------------------

# 36. Security Headers

For web applications, evaluate appropriate headers such as:

-   Content-Security-Policy
-   Strict-Transport-Security
-   X-Content-Type-Options
-   Referrer-Policy
-   Permissions-Policy
-   frame protection

Use modern standards and application-specific configuration.

Do not blindly add obsolete headers just to make a scanner happy.

------------------------------------------------------------------------

# 37. Environment and Deployment Audit

Determine:

-   how the app is built
-   where it runs
-   how secrets are injected
-   which ports are exposed
-   which services are public
-   which services are internal
-   how TLS terminates
-   how databases are reached
-   how backups work
-   how deployments happen

Check for differences between local and production behavior.

------------------------------------------------------------------------

# 38. Security Audit Workflow

Execute the audit in this order:

### Phase A --- Discovery

1.  Map repository structure.
2.  Detect technologies.
3.  Identify entry points.
4.  Identify sensitive components.
5.  Identify trust boundaries.
6.  Identify deployment architecture.

### Phase B --- Automated checks

Use available native tooling for:

-   dependency vulnerabilities
-   secret scanning
-   static analysis
-   lint/security rules
-   infrastructure scanning
-   container scanning

### Phase C --- Manual review

Manually inspect:

-   authentication
-   authorization
-   tenant isolation
-   business logic
-   payment flows
-   webhooks
-   sensitive APIs
-   file handling
-   external integrations

### Phase D --- Exploit reasoning

For each high-risk finding, determine:

-   attacker capability
-   attack path
-   affected resource
-   required conditions
-   potential impact

### Phase E --- Remediation

Fix safe and well-understood issues.

### Phase F --- Verification

Run:

-   tests
-   builds
-   type checks
-   linting
-   security scanners
-   relevant integration tests

Then review the changed code again.

------------------------------------------------------------------------

# 39. Severity Model

Use this severity model:

## Critical

Examples:

-   remote code execution
-   authentication bypass
-   arbitrary database access
-   arbitrary file access with major impact
-   exposed production credentials
-   catastrophic tenant isolation failure
-   payment manipulation with major financial impact

## High

Examples:

-   privilege escalation
-   IDOR/BOLA exposing sensitive data
-   SQL injection
-   SSRF with meaningful internal access
-   sensitive credential leakage
-   major authorization failure

## Medium

Examples:

-   stored XSS with limited impact
-   missing rate limiting on sensitive endpoints
-   excessive data exposure
-   weak session configuration
-   insecure file handling with limited impact

## Low

Examples:

-   missing defense-in-depth controls
-   minor security misconfiguration
-   unnecessary information disclosure
-   outdated but non-vulnerable dependency

## Informational

Best-practice improvements without a direct demonstrated vulnerability.

------------------------------------------------------------------------

# 40. Finding Format

Every finding should use this format:

``` markdown
### [SEVERITY] Finding Title

**Status:** Confirmed / Likely / Potential

**Location:**
`path/to/file.ext:line`

**Issue:**
Clear explanation of what is wrong.

**Why it matters:**
Explain the realistic security impact.

**Attack scenario:**
Describe how an attacker could abuse it.

**Evidence:**
Reference the relevant implementation/configuration.

**Recommended fix:**
Explain the correct remediation.

**Action taken:**
Describe exactly what was changed.

**Verification:**
Explain how the fix was tested.

**Residual risk:**
Mention anything that still requires attention.
```

------------------------------------------------------------------------

# 41. Safe Remediation Rules

When fixing issues:

### Automatically fix when:

-   the fix is localized
-   behavior is well understood
-   the change is low risk
-   tests can validate it
-   the security improvement is clear

Examples:

-   removing committed secrets from active code
-   enabling safe security headers
-   fixing obvious authorization checks
-   adding input validation
-   removing debug mode
-   replacing unsafe query construction
-   correcting insecure cookie flags

### Request confirmation before:

-   deleting large amounts of code
-   changing database schema destructively
-   rotating production credentials
-   changing production infrastructure
-   changing authentication architecture
-   changing payment behavior
-   deleting dependencies with unclear usage
-   making breaking API changes
-   modifying data or migrations destructively

If confirmation is unavailable, prepare the exact remediation and
clearly mark it as pending.

------------------------------------------------------------------------

# 42. Do Not Introduce Security Theater

Avoid changes that merely make a scanner happy.

Examples:

-   unnecessary CORS restrictions that break the application
-   arbitrary rate limits
-   useless security headers
-   disabling legitimate functionality
-   replacing working cryptography without reason
-   hiding errors without fixing the underlying issue
-   adding frontend-only validation
-   adding route guards while leaving APIs unprotected

Every security control should have a reason.

------------------------------------------------------------------------

# 43. Verification Requirements

After remediation:

1.  Run the project's existing tests.
2.  Run build/type checks.
3.  Run linting where available.
4.  Run security tooling where available.
5.  Re-check modified security controls.
6.  Search for duplicate vulnerable patterns.
7.  Confirm the intended behavior still works.

If a security fix changes an API, test:

-   valid request
-   invalid request
-   unauthenticated request
-   unauthorized request
-   cross-user request
-   cross-tenant request where applicable
-   malformed input
-   boundary values

------------------------------------------------------------------------

# 44. Final Audit Report

At the end, produce a concise professional report.

Use this structure:

``` markdown
# Security Audit Report

## Executive Summary

Short description of the project's security posture.

## Project Profile

- Application type:
- Technologies:
- Authentication:
- Authorization:
- Database:
- Infrastructure:
- Deployment:
- External services:

## Risk Summary

| Severity | Count |
|---|---:|
| Critical | 0 |
| High | 0 |
| Medium | 0 |
| Low | 0 |
| Informational | 0 |

## Critical / High Findings

List the most important issues first.

## Medium / Low Findings

List remaining issues.

## Security Controls Verified

- [x] Secrets protection
- [x] Authentication
- [x] Authorization
- [x] Input validation
- [x] API security
- [x] Database security
- [x] Dependency security
- [x] File handling
- [x] CORS
- [x] Security headers
- [x] Rate limiting
- [x] Error handling
- [x] Logging
- [x] Infrastructure
- [x] CI/CD
- [x] Git security

Only mark a control as verified when it was actually checked.

## Changes Made

List files and security improvements.

## Verification Performed

List tests, builds, scanners, and checks executed.

## Remaining Risks

List unresolved items and why they remain unresolved.

## Recommended Next Steps

Prioritize the remaining work.

## Overall Security Assessment

Give an overall assessment:

- Critical risk
- High risk
- Moderate risk
- Low risk
- Good security posture

Explain the reasoning briefly.
```

------------------------------------------------------------------------

# 45. Important Agent Behavior

When using this skill, behave as an experienced security engineer.

Do not:

-   assume the project is secure
-   assume the project is insecure
-   blindly trust documentation
-   blindly trust frontend checks
-   blindly upgrade dependencies
-   make destructive changes without understanding them
-   expose discovered secrets in the final report
-   copy actual credentials into output
-   claim a vulnerability without evidence
-   claim a fix was verified when it was not

Do:

-   inspect
-   trace
-   reason
-   test
-   fix
-   verify
-   document

When reporting a secret, redact it:

``` text
sk_live_************
```

Never print the full credential.

------------------------------------------------------------------------

# 46. Final Principle

The goal is not to produce the longest security checklist.

The goal is to make the actual codebase **harder to compromise**.

Think like this:

``` text
Understand the system
        ↓
Identify trust boundaries
        ↓
Find attack surfaces
        ↓
Trace real data flows
        ↓
Validate security controls
        ↓
Identify exploitable weaknesses
        ↓
Prioritize by real-world impact
        ↓
Fix safely
        ↓
Test the fix
        ↓
Re-audit
        ↓
Report clearly
```

A successful audit should leave the project with:

-   fewer exploitable vulnerabilities
-   stronger authentication
-   stronger authorization
-   safer input handling
-   safer secrets management
-   safer dependencies
-   safer infrastructure
-   better abuse protection
-   verified security controls
-   clear documentation of remaining risk
