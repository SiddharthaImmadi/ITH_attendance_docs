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
