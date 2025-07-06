
# 🧪 Unit Testing vs Integration Testing in Clean Architecture

In Clean Architecture, different layers have different responsibilities — and testing strategy must align with that.

---

## 🔍 What’s the Difference Between Unit Test and Integration Test?

| Feature       | Unit Test                                   | Integration Test                           |
|---------------|---------------------------------------------|---------------------------------------------|
| Purpose       | Test a small unit of code in isolation       | Test how components work together           |
| Speed         | Very fast ⚡                                 | Slower 🐢 (uses real dependencies)          |
| Dependency    | No real dependencies (uses mocks/stubs)      | Often uses real services (e.g., database)   |
| Use           | Verify internal logic                        | Verify interaction between parts            |

---

## 🧱 Clean Architecture Layers and Their Recommended Tests

### ✅ Domain Layer → **Unit Tests**

- Contains pure business logic (Entities, ValueObjects)
- Should be fully isolated and fast

```csharp
[Fact]
public void Order_Should_Calculate_Total_Price()
{
    var order = new Order();
    order.AddItem("Item1", 2, 50);

    order.TotalPrice.Should().Be(100);
}
```

---

### ✅ Application Layer → **Unit Test + Integration Test**

- Use **Unit Test** when mocking repositories/services
- Use **Integration Test** to verify full workflows (e.g., saving to DB)

---

### ✅ Infrastructure Layer → **Integration Test**

- Deals with external systems (DB, Email, File, etc.)
- Should be tested using **real or test versions** of those systems

---

### ✅ API Layer → **Integration / End-to-End Test**

- Test API endpoints using TestServer or WebApplicationFactory
- Ensures API works correctly with underlying Application logic

---

## 📌 Decision Guide: When to Use Each

| Situation                         | Test Type         | Reason                              |
|----------------------------------|-------------------|-------------------------------------|
| Business logic in Entity         | ✅ Unit Test       | No dependencies, pure logic         |
| UseCase with mocked repository   | ✅ Unit Test       | Fast and isolated                   |
| UseCase writing to DB            | ✅ Integration     | Need real DB or in-memory           |
| API controller + use case        | ✅ Integration     | Full flow test                      |
| Infrastructure code (EF, email)  | ✅ Integration     | Needs real resource                 |

---

## ✅ Summary Table by Layer

| Layer         | Recommended Tests                         |
|---------------|-------------------------------------------|
| Domain        | ✅ Unit Test                               |
| Application   | ✅ Unit + ✅ Integration Test (as needed)  |
| Infrastructure| ✅ Integration Test                        |
| API           | ✅ Integration / End-to-End Test           |

---

Keep unit tests fast and focused on logic. Use integration tests to verify collaboration between parts. Choose the right test for the right layer.
