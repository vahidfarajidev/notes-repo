# Understanding JWT and Base64URL

## What Is a JWT?

A JWT (JSON Web Token) consists of three parts:

```
HEADER.PAYLOAD.SIGNATURE
```

Each part is Base64URL-encoded — not encrypted — and separated by dots. This structure makes it easy to transmit in HTTP headers and URLs.

---

## Is Base64URL Secure?

**No, Base64URL is not secure on its own.** It is just an encoding method to make data URL-safe and compact.

Let’s break down this example JWT:

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### 🔹 Header (Base64URL-decoded):

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

### 🔹 Payload (Base64URL-decoded):

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "iat": 1516239022
}
```

Both of these parts can be decoded by anyone — even without the secret key.

---

## Where’s the Security Then?

Security comes from the **signature** part:

```
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

This is created using:

- Header + Payload
- A secret key
- A hashing algorithm (e.g., HMAC SHA-256)

When the server receives a JWT, it re-generates the signature using the same secret key and compares it with the token's signature. If they don’t match, the token is **rejected**.

---

## Key Points

- ✅ JWT content is **readable** via Base64URL.
- ❌ JWT is **not encrypted** by default.
- 🔐 JWT security relies on the **signature** and **secret key**.
- ⚠️ Do **not** put sensitive data (e.g., passwords) in the payload.
- ✅ Always use HTTPS to prevent token theft.
- 🔐 For confidentiality, use **JWE (JSON Web Encryption)** in addition to JWT.

---

## Conclusion

JWTs are not meant to hide information — they are meant to **prove authenticity**. While the contents are readable, the signature ensures they haven’t been tampered with.

Always handle JWTs securely and understand what they are — and are not — designed to do.

---

**Resources:**

- [https://jwt.io](https://jwt.io)
- [RFC 7519 - JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
