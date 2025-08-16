# Exception Handling Across Layers in a C# Web API

## 1️⃣ Domain Layer

**Purpose:** Maintain state and enforce business rules.

**Exceptions:**

- **DomainException:** Thrown when a domain rule is violated, e.g.:
  - `InsufficientFundsException` → when the account balance is insufficient.
  - `InvalidAmountException` → when a deposit or withdrawal amount is invalid.
- **Parameter validation exceptions:** Thrown for invalid input parameters, e.g.:
  - `ArgumentException`
  - `ArgumentNullException`

**Note:** System-level exceptions (e.g., database or I/O errors) generally do not occur in the Domain layer. The Domain focuses on enforcing business rules and validating inputs.

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
- So the idea is that SystemExceptions bubble up to the UI layer, where they are ultimately handled and logged—unless there is a specific reason to catch them earlier in the stack.

**Professional note:** In advanced architectures like **CQRS** or large-scale projects, logging is usually **not done in this layer**. Middleware only maps exceptions to responses, while centralized logging is handled elsewhere for consistency and observability.

# Professional Tip: Input Validation and Service Layer Responsibilities

- **Domain is the source of truth:** All core business rules and validations should be enforced inside the Domain.  
  *Example:* `Account.Withdraw` ensures that the balance is sufficient and the amount is positive.

- **Service orchestration:** The Service layer's main responsibility is to coordinate multiple Repositories or Aggregates and manage transaction logic or retry mechanisms.

- **Optional validation:** Input validation in the Service layer is only necessary if the operation involves multiple Aggregates or Repositories and the Domain cannot fully enforce it.

- **Avoid duplicate checks:** Re-validating inputs already checked by the Domain leads to redundant code and unnecessary complexity.

**Summary:** The Service layer **trusts the Domain to validate inputs** and focuses on executing complex scenarios and multi-step transactions.

