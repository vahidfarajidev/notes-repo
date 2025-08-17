# Exception Handling Across Layers in a C# Web API

# 1️⃣ Domain Layer

**Purpose:**  
Maintain entity state and enforce business rules. Domain entities and aggregates ensure consistency and prevent violations of domain rules.

**Exceptions:**  
- **DomainException:** Thrown when a domain rule is violated, e.g.:  
    - `InsufficientFundsException` → when the account balance is insufficient.  
    - `InvalidAmountException` → when a deposit or withdrawal amount is invalid.  
    - `AccountNotFoundException` → when an account does not exist.  
- **Input validation exceptions:** All input validation uses domain-specific exceptions, not system exceptions.

**Note:**  
System-level exceptions (e.g., database or I/O errors) generally do not occur in the Domain layer. The Domain focuses on enforcing business rules and validating inputs.

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

Exactly, according to **DDD and layered architecture principles**, putting a `try-catch` and directly throwing `SqlException` inside the Repository is generally **not recommended**.

### Reasons:

1️⃣ **Repository = Data Access Only**

- The Repository's responsibility is solely to read/write data.
- It should not decide how errors are handled or throw domain-specific exceptions.
- Catching and wrapping a `SqlException` inside the Repository would introduce part of error handling logic into the Infrastructure layer, which is not correct from a DDD perspective.

2️⃣ **System Exceptions = Handled by Application Layer**

- Infrastructure-related exceptions like:
  - `SqlException`
  - `TimeoutException`
  - `DbUpdateException`
- Should be allowed to **bubble up to the Application Layer**, which can then decide whether to retry, fallback, or show an appropriate user message.

3️⃣ **Exceptions in Repository Only in Special Cases**

- Only in cases where you really want to wrap or log the exception, you can use `try-catch`, but:
  - The original exception should still bubble up.
  - The Repository should **not embed domain logic or throw custom domain exceptions**.

💡 **Summary:**

- No, the Repository should **not throw `SqlException` itself**.
- Let the system exceptions bubble up naturally, and let the **Application/Domain Layer** decide how to handle them.


---

## 3️⃣ Service Layer

**Purpose:** Execute business logic and coordinate multiple repositories.

**Exceptions:**

- Mostly **DomainException**, e.g., when a combined business rule is violated.  
- Typically **does not catch SystemException**, unless implementing retry or fallback mechanisms.

# Application Layer Exception Handling & Logging Behavior

### 1️⃣ Is it correct **not to put try/catch in the Application layer**?

✔ **Yes, in this scenario it is correct**, because:

* The **Application layer** is responsible for executing the Use Case and enforcing Domain Rules, not for handling infrastructure exceptions like database failures.
* Exceptions such as `DbUpdateException` are **infrastructure-level errors** and are better handled and logged by **Middleware or Pipeline behaviors**.
* Adding try/catch in the Application layer for all database exceptions would make the code cluttered with boilerplate and could lose important log details.
* Instead, the exception is allowed to bubble up so that the Middleware decides:
  * What message to show to the user
  * What details to log

💡 **Principle:** The Application layer should handle **Use Case-specific exceptions** (e.g., `InvalidAmountException` or `AccountNotFoundException`) but let infrastructure exceptions propagate to Middleware for proper handling.

---

### 2️⃣ Will these exceptions be logged in the Logging Behavior?

✅ **Yes, if you have a proper MediatR pipeline/Behavior logging setup**, these exceptions will typically be logged.

* In a MediatR Logging Behavior:

  * The request is logged **before** the handler executes.
  * The response or exception is logged **after** the handler executes.

* Exceptions thrown in the Application layer (e.g., `DbUpdateException`) are **caught by the Logging Behavior**, logged, and then rethrown.
* The HTTP Middleware can then produce an appropriate **500 Internal Server Error response**.

💡 **Note:** There are effectively two layers of logging:

1. **Pipeline/Behavior logs** – mainly for internal debugging and detailed information.
2. **Middleware logs** – for HTTP requests, monitoring, and user/system reporting.

---

### Step-by-step Exception Flow

1. **Application layer without try/catch**

   * If a handler like `TransferCommandHandler` throws an exception such as `DbUpdateException` and does not catch it, the exception propagates up to the **MediatR pipeline**.

2. **MediatR Logging Behavior**

   * A Logging Behavior wraps the handler:

     ```text
     LoggingBehavior -> TransferCommandHandler -> LoggingBehavior
     ```

   * When the handler throws an exception, the Behavior can **catch it, log it, and rethrow**.
   * Even without a try/catch in the handler, the exception is still logged.

3. **Middleware / UI Layer**

   * After passing through the Behavior, the exception reaches the Middleware.
   * Middleware processes the exception and produces a suitable **HTTP response** (e.g., 500 Internal Server Error).

✅ **Conclusion:** Even without try/catch in the Application layer, exceptions are still logged by the Behavior and propagated upwards.  

---

## 4️⃣ Controller / Middleware (UI Layer)

**Purpose:** Receive requests, call the Service, and return responses to the client.

**Exceptions:**

- Any **DomainException** or **SystemException** bubbling up from Service or Repository layers.  
- Middleware catches these exceptions and converts them to proper HTTP responses (e.g., 400 for domain violations, 500 for system errors).
- So the idea is that SystemExceptions bubble up to the UI layer, where they are ultimately handled and logged—unless there is a specific reason to catch them earlier in the stack.

**Professional note:** In advanced architectures like **CQRS** or large-scale projects, logging is usually **not done in this layer**. Middleware only maps exceptions to responses, while centralized logging is handled elsewhere for consistency and observability.

# Professional Tip: Input Validation and Service Layer Responsibilities

## Domain is the source of truth
All core business rules and validations should be enforced inside the Domain.  
**Example:** `Account.Withdraw` ensures that the balance is sufficient and the amount is positive. The Domain is responsible for all invariant protection and business rules.

## Service orchestration
The Service layer’s main responsibility is to coordinate multiple Repositories or Aggregates, manage transaction logic, and handle retries.  
It may also perform **fail-fast input validation** for obvious external errors (e.g., null IDs, negative amounts, or incorrect formats) **before sending inputs to the Domain**.

## Optional validation
Input validation in the Service layer is only necessary to provide early feedback or when the operation involves multiple Aggregates or Repositories and the Domain cannot fully enforce it.

## Dual validation is not duplication
Even if the Service layer validates inputs, the Domain must **always re-check invariants** to protect its internal state. This ensures safety when the Domain is invoked from multiple paths.

## Summary
- The Service layer focuses on coordinating operations and handling multi-step transactions.  
- The Domain layer enforces all business rules and ensures invariants.  
- Simple input validation can occur in both layers: **Service for fail-fast feedback, Domain for invariant protection**. This is complementary, not redundant.

Further explanation:

# Service vs Domain Layer in DDD

## 1. Service Layer (Fail Fast / Pre-filter)

- This layer receives input from users or external systems.
- Its job is to **quickly catch obvious/simple mistakes** and stop further processing.

Examples:
- Amount cannot be negative
- ID cannot be null
- Date format must be correct

**Main benefit:** provides fast feedback to the user and prevents wasting system resources.

> Note: The focus here is on **efficiency and UX**, not complex business rules.

## 2. Domain Layer (Protecting Invariants / Business Rules)

- This layer contains the core business model and logic.
- Even if the Service Layer has already checked the input, the **Domain still validates it again**.

**Reason:** The Domain must always ensure the **internal state is valid**, since inputs might come from other sources or the Service might not enforce all rules.

Examples:
- Order amount cannot be negative (Invariant)
- Total number of orders should not exceed a certain limit
- Inventory must always be >= 0

> Note: The focus here is on **maintaining business rules and data consistency**, not just fast feedback.

## ✅ Summary

- **Service Layer:** Fail Fast, pre-filter, better UX, prevent unnecessary processing.
- **Domain Layer:** Protect invariants, ensure internal model correctness, guard against unwanted input.

**Conclusion:** These two layers are **complementary, not redundant**.

> Optional: A practical coding example in C# or Java can illustrate how these layers interact.
