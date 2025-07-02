# Strategy Pattern in C#

The **Strategy Pattern** is a behavioral design pattern that allows you to define a family of algorithms, 
put each of them in a separate class, and make their objects interchangeable.

It promotes **open/closed principle**: open for extension, closed for modification.

---

## ✅ When to Use

- You have multiple ways to perform an operation (e.g., different discount types, payment methods)
- You want to change behavior at runtime without modifying the client code

---

## 🧱 Basic Structure

- **Context**: Uses a Strategy to perform a task
- **Strategy (Interface)**: Common interface for all supported algorithms
- **Concrete Strategies**: Different implementations of the strategy

---

## 🧪 Example: Discount Calculation

### 1. Define the Strategy Interface

```csharp
public interface IDiscountStrategy
{
    decimal ApplyDiscount(decimal totalAmount);
}
```

### 2. Create Concrete Strategies

```csharp
public class NoDiscount : IDiscountStrategy
{
    public decimal ApplyDiscount(decimal totalAmount) => totalAmount;
}

public class PercentageDiscount : IDiscountStrategy
{
    private readonly decimal _percent;
    public PercentageDiscount(decimal percent) => _percent = percent;

    public decimal ApplyDiscount(decimal totalAmount)
    {
        return totalAmount - (totalAmount * _percent / 100);
    }
}
```

### 3. Create the Context

```csharp
public class ShoppingCart
{
    private IDiscountStrategy _discountStrategy;

    public ShoppingCart(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public void SetDiscountStrategy(IDiscountStrategy discountStrategy)
    {
        _discountStrategy = discountStrategy;
    }

    public decimal Checkout(decimal totalAmount)
    {
        return _discountStrategy.ApplyDiscount(totalAmount);
    }
}
```

### 4. Usage

```csharp
var cart = new ShoppingCart(new NoDiscount());
Console.WriteLine(cart.Checkout(100)); // Output: 100

cart.SetDiscountStrategy(new PercentageDiscount(10));
Console.WriteLine(cart.Checkout(100)); // Output: 90
```

---

## 🧠 Summary

| Role         | Responsibility                          |
|--------------|------------------------------------------|
| Strategy     | Interface defining the algorithm         |
| Concrete     | Classes implementing the algorithm       |
| Context      | Uses the strategy to perform the action  |

✅ This pattern makes it easy to **add new behaviors** (like new types of discounts) without changing existing code.

---

## ✅ Real-World Analogy

> Think of a shopping app where users can have different types of discounts:  
> no discount, seasonal percentage, or coupon code.  
> Each strategy is plug-and-play without modifying checkout logic.
