# backend_authentication.md — JWT & Password Security

> How authentication and authorization work in the backend: JWT tokens, password hashing, token validation, and security best practices.

## 1. Authentication Flow (Theory)

```
┌──────────────┐
│   Member     │
└──────┬───────┘
       │ POST /auth/login (email, password)
       ▼
┌──────────────────────────────────────┐
│  Backend: Verify Password            │
│  1. Look up user by email            │
│  2. Hash submitted password          │
│  3. Compare with stored hash         │
│  4. If match, generate JWT           │
└──────┬───────────────────────────────┘
       │ Response 200 (JWT + user profile)
       ▼
┌──────────────┐
│   Member     │ Stores JWT in localStorage
└──────┬───────┘
       │ GET /me (with JWT in Authorization header)
       ▼
┌──────────────────────────────────────┐
│  Backend: Verify JWT                 │
│  1. Extract token from header        │
│  2. Decode token (verify signature)  │
│  3. Extract user_id from payload     │
│  4. Return user profile              │
└──────┬───────────────────────────────┘
       │ Response 200 (user profile)
       ▼
┌──────────────┐
│   Member     │ Logged in, can make requests
└──────────────┘
```

## 2. JWT Structure

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJ1dWlkLWhlcmUiLCJleHAiOjE2MjMxMjM0NTZ9.signature-here
  ▲                                         ▲                                          ▲
  Header                                    Payload                                    Signature
  (alg, type)                               (user_id, exp, iat)                       (HMAC-SHA256)
```

**Header:**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**
```json
{
  "sub": "8a1b2c3d-4444-4a5b-9c6d-abcdef654321",  // user_id (sub = subject)
  "exp": 1623123456,                               // expiration timestamp (Unix time)
  "iat": 1623119856,                               // issued at timestamp
  "email": "member1@innotech-hub.com"              // optional, for convenience
}
```

**Signature:** Generated using `HMAC-SHA256(header.payload, JWT_SECRET_KEY)`

## 3. Password Hashing (Backend Implementation)

**Never store plaintext passwords.** Use `passlib` with bcrypt or argon2.

```python
# app/core/security.py
from passlib.context import CryptContext

# Configure password hashing
pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def hash_password(password: str) -> str:
    """Hash a plaintext password."""
    return pwd_context.hash(password)

def verify_password(plaintext: str, hashed: str) -> bool:
    """Verify plaintext password against hash."""
    return pwd_context.verify(plaintext, hashed)
```

**Hash properties:**
- Slow by design (intentional computational cost)
- Random salt embedded (same password ≠ same hash)
- Cannot reverse (one-way)

**Example:**
```python
# Registration or password reset
plaintext_password = "MemberPass123!"
stored_hash = hash_password(plaintext_password)
# stored_hash = "$2b$12$R9h2cIPzXVsaASddNl.Yt.EHhpzqYdj3GHqFJFkIp5ylpVT/cM..." (different each time)

# Login verification
if verify_password(submitted_password, stored_hash):
    # Correct password
else:
    # Incorrect password
```

## 4. JWT Token Generation (Backend Implementation)

```python
# app/core/security.py
from datetime import datetime, timedelta, timezone
import jwt

JWT_SECRET_KEY = os.getenv("JWT_SECRET_KEY")
JWT_ALGORITHM = "HS256"
JWT_EXPIRE_MINUTES = 60

def create_access_token(user_id: str, email: str, expires_delta: timedelta = None) -> str:
    """Create a JWT access token."""
    if expires_delta is None:
        expires_delta = timedelta(minutes=JWT_EXPIRE_MINUTES)
    
    expire = datetime.now(timezone.utc) + expires_delta
    payload = {
        "sub": user_id,
        "email": email,
        "exp": expire,
        "iat": datetime.now(timezone.utc)
    }
    
    token = jwt.encode(payload, JWT_SECRET_KEY, algorithm=JWT_ALGORITHM)
    return token
```

**Usage in login endpoint:**
```python
@router.post("/auth/login")
async def login(credentials: LoginRequest, db: AsyncSession = Depends(get_db)):
    user = await db.query(User).filter(User.email == credentials.email).first()
    
    if not user or not verify_password(credentials.password, user.password_hash):
        raise HTTPException(status_code=401, detail="INVALID_CREDENTIALS")
    
    # Generate token
    access_token = create_access_token(str(user.id), user.email)
    
    return {
        "access_token": access_token,
        "token_type": "bearer",
        "expires_in_minutes": JWT_EXPIRE_MINUTES,
        "user": {
            "id": user.id,
            "full_name": user.full_name,
            "email": user.email,
            "role": user.role
        }
    }
```

## 5. JWT Token Verification (Backend Implementation)

```python
# app/core/security.py
from fastapi import Depends, HTTPException
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/login")

def verify_token(token: str) -> dict:
    """Verify JWT signature and expiration, return payload."""
    try:
        payload = jwt.decode(token, JWT_SECRET_KEY, algorithms=[JWT_ALGORITHM])
        user_id = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=401, detail="UNAUTHORIZED")
        return payload
    except jwt.ExpiredSignatureError:
        raise HTTPException(status_code=401, detail="UNAUTHORIZED")  # Token expired
    except jwt.InvalidTokenError:
        raise HTTPException(status_code=401, detail="UNAUTHORIZED")  # Invalid signature
```

**Usage in endpoints (dependency injection):**
```python
async def get_current_user(token: str = Depends(oauth2_scheme), db: AsyncSession = Depends(get_db)) -> User:
    """Dependency to get the current authenticated user."""
    payload = verify_token(token)
    user_id = payload.get("sub")
    
    user = await db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=401, detail="UNAUTHORIZED")
    
    return user

# In any protected endpoint:
@router.get("/me")
async def get_me(current_user: User = Depends(get_current_user)):
    return current_user
```

## 6. Authorization (Role-Based Access)

After authentication, verify the user has the right role:

```python
async def get_current_admin(current_user: User = Depends(get_current_user)) -> User:
    """Dependency to ensure current user is admin."""
    if current_user.role != "admin":
        raise HTTPException(status_code=403, detail="FORBIDDEN")
    return current_user

# In endpoint:
@router.post("/sessions")
async def create_session(req: SessionCreate, current_user: User = Depends(get_current_admin)):
    # current_user is guaranteed to be admin
    ...
```

## 7. Token Expiration & Refresh

**Phase 1:** Tokens expire after 60 minutes. No refresh token.

**How expiration works:**
- Token contains `exp` (expiration time in Unix seconds)
- On every request, `verify_token()` checks: `if payload['exp'] < now: raise ExpiredSignatureError`
- Member logs out → token becomes invalid (frontend deletes from localStorage)

**User experience:**
- Login, get 60-min token
- Work for up to 60 minutes
- After 60 min of inactivity, token expires
- Next request gets 401 → frontend redirects to login
- No "remember me" or background refresh in Phase 1

## 8. Security Best Practices

| Practice | Implementation |
|---|---|
| Never log passwords or tokens | Use `exclude_keys=["password", "token"]` in logging |
| Use HTTPS only (production) | Local dev uses HTTP; production must enforce TLS |
| Store JWT_SECRET_KEY in env, never in code | `JWT_SECRET_KEY = os.getenv("JWT_SECRET_KEY")` |
| Hash passwords with strong algorithm | bcrypt with cost factor 12+ |
| Validate token on every protected request | `get_current_user` is a dependency on all secured endpoints |
| Set short expiration times | 60 minutes for Phase 1 |
| Return generic error messages | "Incorrect email or password" (not "email not found") |

## 9. Token Storage (Frontend Concern)

Frontend receives JWT and stores it in **localStorage**. On every API request:
```javascript
const token = localStorage.getItem("access_token");
fetch("/me", {
    headers: {
        "Authorization": `Bearer ${token}`
    }
});
```

Backend extracts token from `Authorization: Bearer <token>` header.

## 10. Session Management

**Stateless:** Server doesn't store sessions. Every request includes the token.

**Logout (Frontend):**
- Delete token from localStorage
- Frontend redirects to login
- Token still valid on server for 60 min, but frontend won't use it

**Logout (Backend):** Not implemented in Phase 1. Token validity is time-based only.

## 11. Environment Variables Required

```
JWT_SECRET_KEY=<random string, min 32 chars>
JWT_ALGORITHM=HS256
JWT_EXPIRE_MINUTES=60
```

Generate secret: `python -c "import secrets; print(secrets.token_urlsafe(32))"`

## 12. Common Attacks & Mitigations

| Attack | How it works | Mitigation |
|---|---|---|
| Brute force login | Try 1000s of password combinations | Rate limit /auth/login endpoint |
| Token hijacking | Steal JWT from localStorage | HTTPS only, short expiration |
| Token replay | Use stolen token multiple times | Token has expiration timestamp |
| SQL injection on password | Inject SQL in password field | Use parameterized queries (SQLAlchemy does this) |
| Password exposure in logs | Log the password field | Never log password fields |


## 13. Phase 2 Authentication & Authorization Extensions

Phase 2 continues using the existing JWT-based authentication system.

No changes are required to:

- password hashing;
- JWT generation;
- JWT verification;
- token expiration;
- login flow.

The focus of Phase 2 is expanding authorization rules for the new attendance lifecycle.

---

## 13.1 Authentication Flow

Authentication remains unchanged.

```
Login

↓

JWT Issued

↓

Protected API Request

↓

JWT Verification

↓

Role Verification

↓

Business Logic
```

All Phase 2 endpoints continue to require a valid JWT.

---

## 13.2 Member Permissions

Authenticated members may:

- view available sessions;
- check in;
- view active attendance;
- submit early check-out requests;
- view approval status;
- complete approved check-out;
- view attendance history;
- view attendance summaries.

Members may not:

- approve requests;
- reject requests;
- access monitoring endpoints;
- access another member's attendance.

---

## 13.3 Administrator Permissions

Authenticated administrators may:

- manage sessions;
- monitor live attendance;
- review early check-out requests;
- approve requests;
- reject requests;
- generate reports;
- access monitoring statistics.

Administrators may only manage resources they own where ownership rules apply.

---

## 13.4 Authorization Dependencies

Continue using dependency injection.

Examples:

```python
get_current_user()

get_current_admin()
```

Business services should assume authorization has already been validated by the route layer.

---

## 13.5 Endpoint Protection

Member endpoints:

```
GET  /attendance/active

GET  /attendance/summary

GET  /attendance/timeline

POST /attendance/check-out

POST /attendance/early-checkout

GET  /attendance/approval-status
```

Administrator endpoints:

```
GET  /monitoring/live

GET  /monitoring/statistics

GET  /monitoring/pending-requests

POST /attendance/approve

POST /attendance/reject
```

All endpoints require authentication.

Administrator endpoints additionally require administrator authorization.

---

## 13.6 Ownership Validation

Authorization extends beyond roles.

Services should verify ownership before returning protected resources.

Examples:

Members:

- may only access their own attendance records.

Administrators:

- may only manage sessions they created.

Ownership validation belongs in the service layer.

---

## 13.7 Security Principles

Phase 2 continues the existing security model.

Principles:

- authenticate every protected request;
- authorize every sensitive operation;
- never trust client-provided user identifiers;
- derive user identity from the verified JWT;
- validate ownership before returning data.

---

## 13.8 Error Responses

Unauthorized authentication:

```
401 UNAUTHORIZED
```

Insufficient permissions:

```
403 FORBIDDEN
```

Ownership violation:

```
404 NOT_FOUND
```

Returning `404` for unauthorized ownership prevents resource enumeration.

---

## 13.9 Testing Requirements

Verify:

- members cannot access administrator endpoints;
- administrators can approve requests;
- members cannot approve requests;
- members cannot access another member's attendance;
- administrators cannot manage sessions they do not own;
- JWT authentication remains required for all protected endpoints.

---

## 13.10 Phase 2 Security Principles

The authentication architecture remains unchanged.

Phase 2 extends authorization while preserving:

- stateless JWT authentication;
- role-based access control;
- ownership validation;
- secure password handling;
- short-lived access tokens;
- dependency-based authorization.

# 14. Phase 3 Authentication & Authorization Extensions

Phase 3 continues using the existing JWT authentication architecture.

No changes are required to:

- password hashing;
- JWT generation;
- JWT verification;
- token expiration;
- login flow.

Phase 3 extends authorization rules to support activity management, volunteer assignments, activity reviews, and template management.

---

## 14.1 Authentication Flow

Authentication remains unchanged.

```
Login

↓

JWT Issued

↓

Protected API Request

↓

JWT Verification

↓

Role Verification

↓

Ownership Validation

↓

Business Logic
```

All Phase 3 endpoints require a valid JWT.

---

## 14.2 Member Permissions

Authenticated members may:

- view assigned activities;
- view activity details;
- create progress updates;
- upload activity evidence;
- submit activities for review;
- view their own review history;
- view activity history.

Members may not:

- create activities;
- publish activities;
- cancel activities;
- archive activities;
- assign volunteers;
- review activities;
- manage activity templates.

---

## 14.3 Administrator Permissions

Authenticated administrators may:

- create activities;
- update activities;
- publish activities;
- cancel activities;
- archive activities;
- assign volunteers;
- review submitted activities;
- request changes;
- verify completed activities;
- manage activity templates;
- generate activity reports.

Administrators may only manage resources they own where ownership rules apply.

---

## 14.4 Authorization Dependencies

Continue using dependency injection.

Examples:

```python
get_current_user()

get_current_admin()
```

Business services should assume authorization has already been validated by the route layer.

---

## 14.5 Endpoint Protection

Member endpoints:

```
GET  /members/me/assignments

GET  /assignments/{id}/progress

POST /assignments/{id}/progress

POST /assignments/{id}/submit

GET  /members/me/activity-history
```

Administrator endpoints:

```
POST  /activities

PATCH /activities/{id}

POST  /activities/{id}/publish

POST  /activities/{id}/cancel

POST  /activities/{id}/archive

POST  /activities/{id}/assign

GET   /activities/review/pending

POST  /reviews/{id}/verify

POST  /reviews/{id}/needs-changes

POST  /activity-templates

PATCH /activity-templates/{id}

POST  /activity-templates/{id}/apply
```

All endpoints require authentication.

Administrator endpoints additionally require administrator authorization.

---

## 14.6 Ownership Validation

Authorization extends beyond roles.

Services should verify ownership before returning protected resources.

Examples:

Members:

- may only access assignments assigned to them;
- may only upload progress to their own assignments;
- may only submit their own assignments;
- may only access their own activity history.

Administrators:

- may only manage activities they created;
- may only review activities belonging to events they manage.

Ownership validation belongs in the service layer.

---

## 14.7 Activity Review Authorization

Review operations require administrator authorization.

The backend must verify:

- reviewer is an administrator;
- assignment exists;
- assignment is awaiting review;
- assignment has not already been verified.

Review decisions become permanent records.

---

## 14.8 Template Authorization

Only administrators may:

- create templates;
- modify templates;
- apply templates.

Members have no access to template management endpoints.

---

## 14.9 Security Principles

Phase 3 continues the existing security model.

Principles:

- authenticate every protected request;
- authorize every sensitive operation;
- never trust client-provided user identifiers;
- derive user identity from the verified JWT;
- validate ownership before returning data;
- preserve review history;
- prevent unauthorized activity management.

---

## 14.10 Error Responses

Unauthorized authentication:

```
401 UNAUTHORIZED
```

Insufficient permissions:

```
403 FORBIDDEN
```

Ownership violation:

```
404 NOT_FOUND
```

Invalid activity state:

```
409 CONFLICT
```

Returning `404` for unauthorized ownership prevents resource enumeration.

---

## 14.11 Testing Requirements

Verify:

- members cannot create activities;
- members cannot assign volunteers;
- members cannot review activities;
- members cannot access another member's assignments;
- administrators can manage activities;
- administrators can review submissions;
- administrators can manage templates;
- JWT authentication remains required for all protected endpoints.

---

## 14.12 Phase 3 Security Principles

The authentication architecture remains unchanged.

Phase 3 extends authorization while preserving:

- stateless JWT authentication;
- role-based access control;
- ownership validation;
- secure password handling;
- short-lived access tokens;
- dependency-based authorization;
- centralized authorization logic.

# 15. Phase 4 Authentication & Authorization Extensions

Phase 4 preserves the existing JWT-based authentication architecture while extending backend security for production readiness.

No changes are required to:

- password hashing;
- JWT generation;
- JWT verification;
- JWT payload structure;
- authentication dependencies.

Phase 4 introduces production security capabilities including:

- authentication audit logging;
- login rate limiting;
- device profile monitoring;
- suspicious login detection;
- administrator session revocation;
- production security monitoring.

---

## 15.1 Authentication Flow

The authentication lifecycle is extended with additional security checks.

```
Login Request

↓

Rate Limit Validation

↓

Credential Verification

↓

JWT Generation

↓

Authentication Audit

↓

Device Profile Capture

↓

Suspicious Login Evaluation

↓

Response
```

Authentication continues using stateless JWT tokens.

Security monitoring operates independently from authentication.

---

## 15.2 Login Rate Limiting

To reduce brute-force attacks, login attempts are rate limited.

Policy:

- maximum 5 consecutive failed login attempts;
- account locked for 15 minutes;
- successful login resets the failed-attempt counter;
- administrators may manually unlock accounts.

Rate limiting applies only to authentication endpoints.

---

## 15.3 Device Profile Monitoring

Authentication events capture a lightweight device profile.

Examples include:

- browser information;
- operating system;
- device type;
- user-agent;
- other non-invasive request metadata.

Device profiles are used only for:

- audit history;
- suspicious behaviour analysis;
- administrator investigations.

Device profiles are not used as an authentication factor.

---

## 15.4 Suspicious Authentication Detection

Authentication events are evaluated for suspicious behaviour.

Examples include:

- login from an unfamiliar device;
- repeated spoofing indicators;
- abnormal authentication patterns;
- other production security signals.

When suspicious behaviour is detected:

- login continues;
- a security event is created;
- administrators are notified;
- the event becomes part of the audit history.

The system does not automatically block authentication based solely on suspicious indicators.

---

## 15.5 Authentication Audit Logging

Every authentication-related event generates an audit record.

Examples include:

- successful login;
- failed login;
- account lock;
- administrator unlock;
- token revocation;
- suspicious login detection;
- logout events where applicable.

Audit records remain immutable.

Authentication auditing uses the centralized AuditLogService.

---

## 15.6 JWT Revocation

Phase 4 introduces administrator-controlled JWT revocation.

Administrators may revoke all active sessions belonging to a user.

Typical use cases include:

- compromised accounts;
- suspicious activity;
- security incidents;
- administrative enforcement.

Revoked sessions require users to authenticate again before accessing protected resources.

---

## 15.7 Session Management

Session management continues using stateless JWT authentication.

When administrator revocation occurs:

```
Administrator

↓

Revoke User Sessions

↓

Invalidate All Active Tokens

↓

User Re-authenticates
```

All active sessions for the affected user are revoked together.

Selective per-device revocation is not implemented.

---

## 15.8 Authorization Continuity

Authorization continues using the existing role-based and ownership-based model.

Phase 4 introduces no new user roles.

Existing authorization rules remain unchanged.

Business services continue validating:

- authenticated identity;
- administrator permissions;
- ownership requirements;
- resource access.

---

## 15.9 Security Event Handling

Security-related authentication events follow a consistent workflow.

```
Authentication Event

↓

Security Evaluation

↓

Audit Recording

↓

Administrator Notification

↓

Administrative Review
```

The backend records security events without automatically enforcing account restrictions.

Final enforcement decisions remain under administrator control.

---

## 15.10 Error Responses

Additional authentication responses introduced by Phase 4 include:

```
401 UNAUTHORIZED
```

Invalid or revoked token.

```
403 FORBIDDEN
```

Insufficient permissions.

```
423 LOCKED
```

Account temporarily locked after excessive failed login attempts.

```
429 TOO_MANY_REQUESTS
```

Rate limit exceeded.

Error responses continue following the standardized backend response format.

---

## 15.11 Testing Requirements

Authentication testing should verify:

- login rate limiting;
- automatic account lock after repeated failures;
- administrator account unlock;
- device profile recording;
- authentication audit generation;
- suspicious login detection;
- administrator session revocation;
- revoked token rejection;
- existing authorization rules remain unchanged.

---

## 15.12 Phase 4 Security Principles

Phase 4 extends the authentication architecture while preserving previous security guarantees.

Principles:

- preserve stateless JWT authentication;
- authenticate every protected request;
- authorize every sensitive operation;
- never trust client-provided identity;
- derive identity from verified JWT tokens;
- audit all authentication events;
- detect suspicious behaviour without automatic enforcement;
- maintain administrator-controlled security decisions;
- revoke all active sessions when administrative revocation occurs;
- preserve backward compatibility with previous phases.
