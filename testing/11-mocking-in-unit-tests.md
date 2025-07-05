# 🧪 Mocking Dependencies in Unit Testing (C#)

In unit testing, we often want to **isolate** the unit under test from its external dependencies. That’s where **mocking** comes in.

---

## ❓ What is Mocking?

Mocking is the practice of **replacing real objects with controlled, fake versions** (mocks) in your tests. These mocks simulate the behavior of complex or external systems like:

- Databases
- Web APIs
- File systems
- Services

---

## 🎯 Why Mock?

- ✅ Isolate code for **true unit testing**
- ✅ Avoid external dependencies (like databases)
- ✅ Make tests **faster** and **more reliable**
- ✅ Verify how dependencies are **used**, not just what they return

---

## 🔧 Popular Mocking Frameworks in .NET

| Framework     | Description                                      | Pros                            |
|---------------|--------------------------------------------------|---------------------------------|
| **Moq**       | Most popular, intuitive syntax                   | Lightweight, well-supported     |
| **NSubstitute**| Clean, intuitive for interfaces                 | Friendly syntax, auto-substitutes |
| **FakeItEasy**| Flexible, readable, designed for .NET            | Simple API                      |
| **JustMock**  | Commercial, powerful (also free version)         | Mock non-virtual members        |

> ✅ **Moq** is the most widely used and recommended for most projects.

---

## 🔍 Example with Moq

### Let's say we have this interface:

```csharp
public interface IEmailService
{
    void SendEmail(string to, string subject);
}
```

### And we want to test this class:

```csharp
public class Notification
{
    private readonly IEmailService _emailService;

    public Notification(IEmailService emailService)
    {
        _emailService = emailService;
    }

    public void Notify(string userEmail)
    {
        if (!string.IsNullOrEmpty(userEmail))
        {
            _emailService.SendEmail(userEmail, "Welcome!");
        }
    }
}
```

### ✅ Unit Test using Moq:

```csharp
using Moq;
using Xunit;

public class NotificationTests
{
    [Fact]
    public void Should_Send_Email_When_Email_Is_Valid()
    {
        // Arrange
        var mockEmail = new Mock<IEmailService>();
        var notification = new Notification(mockEmail.Object);

        // Act
        notification.Notify("test@example.com");

        // Assert
        mockEmail.Verify(e => e.SendEmail("test@example.com", "Welcome!"), Times.Once);
    }
}
```

---

## 🧠 Summary

- Mocking allows us to **test in isolation**
- Use tools like **Moq** to create fake versions of services
- You can verify **calls** and **parameters**
- Helps you **decouple** logic from infrastructure

---

Let me know if you'd like to extend this to:
- Async method mocking
- Mocking return values
- Behavior verification
