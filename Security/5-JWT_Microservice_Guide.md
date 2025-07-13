
# ✅ JWT Usage in Microservice Architecture

This document summarizes a real-world implementation of JWT authentication across multiple microservices, focusing on **AuthService** and **AppService** communication, security, and potential vulnerabilities.

---

## 🧪 What Was Implemented

- 🔐 A JWT was generated from `AuthService` (based on roles/claims).
- 🎯 The JWT was manually inserted into Swagger UI of `AppService`.
- 🛡️ A protected endpoint (e.g., `/secure`, `/role`, `/policy`) was accessed.
- ✅ Because `AppService` had proper authentication/authorization setup, access was granted.

---

## 🎯 Understanding the Roles

| Component     | Role Description |
|---------------|------------------|
| **AuthService** | Authenticates users and issues JWT tokens |
| **AppService**  | Resource Server that protects endpoints via `[Authorize]` |
| **Client (React, etc.)** | UI that sends login and token-based requests |

---

## 🔄 Real-World Token Flow (Frontend + Microservices)

1. **Login Request (POST `/auth/login`)**
   - Client sends username/password to `AuthService`.
   - JWT is returned.

2. **Token Storage (Frontend)**
   - JWT saved in `localStorage`, `sessionStorage`, or `HttpOnly Cookie`.

3. **Token Usage**
   - Sent as `Authorization: Bearer <token>` in all protected requests.

4. **AppService Behavior**
   - Validates token signature using `secret key`.
   - Checks `issuer`, `audience`, `expiration`, roles/claims.

```ts
// React Axios Example
const token = localStorage.getItem("token");
await axios.get("https://app.myapp.com/secure/data", {
  headers: { Authorization: `Bearer ${token}` }
});
```

---

## 🔗 Are AppServices Directly Connected to AuthService?

> ❌ No. AppServices **do not** contact AuthService to verify every token.

Instead:

- ✅ AppService validates the token **locally** using the `secret key`.
- If the token’s signature, expiry, and claims are valid → access is allowed.
- There’s **no need to query AuthService again** unless revocation or dynamic checks are needed.

---

## 🔐 What If a Token Is Stolen?

If a valid JWT is intercepted:

- AppService **can't tell** if it's stolen — unless additional checks are done.

### ✅ Security Recommendations

| Threat | Solution |
|--------|----------|
| Token theft | Short `exp` + Refresh Token |
| Replay attack | Include device/IP info in claims |
| Logout/revoke | Use a blacklist (e.g., Redis) |
| Sensitive actions | Require MFA or re-authentication |
| XSS | Use HttpOnly Cookies, not `localStorage` |

---

## 🔍 Token Verification Explained

```csharp
IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey)),
ValidateIssuer = true,
ValidIssuer = "auth.myapp.com"
```

- Token must be signed with a key known by both AuthService and AppService.
- AppService **never trusts token blindly**, it verifies signature + expiration.

---

## 🔑 Symmetric vs Asymmetric Algorithms

| Type | Key Handling | Example |
|------|--------------|---------|
| Symmetric | One shared `secretKey` for both sign and verify | `HS256` |
| Asymmetric | `PrivateKey` (sign) & `PublicKey` (verify) | `RS256` |

In this project → `HS256` is used.

---

## ✅ Summary Diagram

```
React Client
    |
    | POST /auth/login
    V
AuthService --> returns JWT
    |
    | Save token
    V
React → Authorization: Bearer eyJ...
    |
    V
AppService -- validates token
```

---

## ✅ Want More?

Let us know if you want to:

- Add Refresh Token flow
- Implement token blacklist or revocation
- Use RSA instead of HMAC
- Connect gateway + services
