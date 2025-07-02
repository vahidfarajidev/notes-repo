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


---

## 🔄 Why Use `SetDiscountStrategy()` When We Have a Constructor?

The constructor lets you set an initial discount strategy:

```csharp
var cart = new ShoppingCart(new NoDiscount());
```

But `SetDiscountStrategy()` gives you flexibility to **change the strategy at runtime**:

```csharp
cart.SetDiscountStrategy(new PercentageDiscount(10));
```

### ✅ Why is this useful?

Imagine a shopping app:
- A user starts with **no discount**
- Later, they apply a **promo code** or join a **special offer**

Without `SetDiscountStrategy()`, you'd need to recreate the entire cart.  
With it, you can **change behavior on the fly**, without modifying the client code.

### 🧠 Summary:

| Approach            | Benefit                                   |
|---------------------|-------------------------------------------|
| Constructor only     | Simple, fixed behavior                    |
| + `SetDiscountStrategy()` | Flexible — allows runtime behavior change |

This dynamic flexibility is what makes the **Strategy Pattern** so powerful.

---

## 🔍 Changeable vs Extensible — Strategy Pattern Benefits

The Strategy Pattern offers two powerful benefits:

| Feature        | What It Means                                                     |
|----------------|-------------------------------------------------------------------|
| 🔁 **Changeable**   | You can change the behavior **at runtime** using `SetDiscountStrategy()` |
| 🚀 **Extensible**   | You can add **new discount types** without modifying existing code        |

### 🧪 In This Example:

- The `ShoppingCart` can **dynamically switch** between discount strategies  
- You can add a new class like `HolidayDiscount` or `CouponDiscount`  
  — without changing the `ShoppingCart` class itself

This design follows the **Open/Closed Principle**:
> **Open for extension**, but **closed for modification**

✅ Strategy Pattern enables **runtime flexibility** and **codebase scalability**

---

## 🧱 Open for Extension, Closed for Modification — In Practice

This design follows the **Open/Closed Principle**:

> "Software entities should be open for extension, but closed for modification."

### 🧪 In This Example:

- You can add new strategies like `HolidayDiscount` or `CouponDiscount`
- You **don’t need to change** the existing `ShoppingCart` or interface code

### ✅ What’s Open for Extension?

- `IDiscountStrategy`: You can create many new implementations of this interface
- `ShoppingCart`: It can work with **any** strategy passed to it — no changes required

### 🔒 What’s Closed for Modification?

- The `ShoppingCart` class: No need to touch it when adding new behaviors
- Existing concrete strategies like `NoDiscount`, `PercentageDiscount`: We don't edit them, we add new ones

### 📌 Summary:

| Component            | Open for Extension | Closed for Modification |
|----------------------|--------------------|--------------------------|
| `IDiscountStrategy`  | ✅ Yes             | ✅ Yes                   |
| `ShoppingCart`       | ✅ Yes             | ✅ Yes                   |
| Strategy Implementations | ➕ Add new classes | ✖ Don’t modify existing ones |

This is the essence of Strategy Pattern and solid design:  
> **Add behavior without breaking or changing existing logic.**
