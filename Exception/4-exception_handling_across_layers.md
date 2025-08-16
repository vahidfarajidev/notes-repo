# Exception Handling Across Layers in a C# Web API

## 1️⃣ Domain Layer

**Purpose:** Maintain state and enforce business rules.

**Exceptions:**

- **DomainException:** Thrown when a domain rule is violated, e.g.:
  - `InsufficientFundsException` → when the account balance is insufficient.
  - `InvalidAmountException` → when a deposit or withdrawal amount is invalid.
- **SystemException:** Rarely occurs in the Domain layer, only for unexpected runtime errors. The focus of the Domain is on business rules, not database or I/O issues.

---

## 2️⃣ Repository Layer

**Purpose:** Data access (DB, File, External APIs).

**Exceptions:**

- **DomainException:** Thrown when data is missing or domain rules are violated in the stored data (e.g., negative balance in DB that shouldn’t exist).  
- **SystemException:** System or database-related exceptions, such as:
  - `SqlException`
  - `DbUpdateException`
  - `TimeoutException`

**Behavior:** Repository typically does **not catch SystemExceptions**; they are allowed to **bubble up** to higher layers.

---

## 3️⃣ Service Layer

**Purpose:** Execute business logic and coordinate multiple repositories.

**Exceptions:**

- Mostly **DomainException**, e.g., when a combined business rule is violated.  
- Typically **does not catch SystemException**, unless implementing retry or fallback mechanisms.

---

## 4️⃣ Controller / Middleware (UI Layer)

**Purpose:** Receive requests, call the Service, and return responses to the client.

**Exceptions:**

- Any **DomainException** or **SystemException** bubbling up from Service or Repository layers.  
- Middleware catches these exceptions and converts them to proper HTTP responses (e.g., 400 for domain violations, 500 for system errors).

**Professional note:** In advanced architectures like **CQRS** or large-scale projects, logging is usually **not done in this layer**. Middleware only maps exceptions to responses, while centralized logging is handled elsewhere for consistency and observability.
