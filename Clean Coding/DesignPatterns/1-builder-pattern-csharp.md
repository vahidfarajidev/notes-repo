# Builder Pattern in C#

The **Builder Pattern** is a creational design pattern that separates the construction of a complex object 
from its representation — so the same construction process can create different representations.

---

## ✅ When to Use

- When you need to build an object with many optional parameters
- To avoid telescoping constructors (too many overloaded constructors)
- To make object creation more readable and flexible

---

## 🧱 Basic Structure

- **Product**: The complex object being built
- **Builder**: Interface or abstract class with build steps
- **ConcreteBuilder**: Implements the steps
- **Director** (optional): Orchestrates the building steps

---

## 🧪 Example: Building a Meal

### 1. Define the Product

```csharp
public class Meal
{
    public string Main { get; set; }
    public string Side { get; set; }
    public string Drink { get; set; }

    public override string ToString()
    {
        return $"Main: {Main}, Side: {Side}, Drink: {Drink}";
    }
}
```

### 2. Define the Builder Interface

```csharp
public interface IMealBuilder
{
    void BuildMain();
    void BuildSide();
    void BuildDrink();
    Meal GetMeal();
}
```

### 3. Create a Concrete Builder

```csharp
public class VegMealBuilder : IMealBuilder
{
    private Meal _meal = new Meal();

    public void BuildMain() => _meal.Main = "Grilled Tofu";
    public void BuildSide() => _meal.Side = "Salad";
    public void BuildDrink() => _meal.Drink = "Orange Juice";
    public Meal GetMeal() => _meal;
}
```

### 4. Create the Director (Optional)

```csharp
public class MealDirector
{
    private IMealBuilder _builder;

    public MealDirector(IMealBuilder builder)
    {
        _builder = builder;
    }

    public Meal ConstructMeal()
    {
        _builder.BuildMain();
        _builder.BuildSide();
        _builder.BuildDrink();
        return _builder.GetMeal();
    }
}
```

### 5. Usage

```csharp
var builder = new VegMealBuilder();
var director = new MealDirector(builder);
var meal = director.ConstructMeal();

Console.WriteLine(meal); 
// Output: Main: Grilled Tofu, Side: Salad, Drink: Orange Juice
```

---

## 🧠 Summary

| Role           | Responsibility                    |
|----------------|------------------------------------|
| Product        | The object being built             |
| Builder        | Defines the steps to build product |
| ConcreteBuilder| Implements the build steps         |
| Director       | Optional — guides the build process|

✅ The Builder Pattern makes it easier to build complex objects step-by-step and improves code clarity.

---

## ✅ Real-World Analogy

> Think of ordering a custom meal:  
> You can specify the main dish, a side, and a drink — all customized.  
> The builder handles step-by-step construction so the result is always valid and consistent.
