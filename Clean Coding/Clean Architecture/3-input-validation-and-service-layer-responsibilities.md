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
