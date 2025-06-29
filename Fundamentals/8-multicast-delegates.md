# Multicast Delegates in C#

In C#, delegates allow you to reference methods as variables. A powerful feature of delegates is **multicast invocation**, where you can combine multiple methods into a single delegate and execute them in order.

This is particularly useful for scenarios like event handling or when you want to trigger a sequence of actions with a single call.

## Example: Multicast Delegate

```csharp
public delegate void Notify();

void SayHello() => Console.WriteLine("Hello");
void SayBye() => Console.WriteLine("Bye");

Notify notify = SayHello;
notify += SayBye;

notify(); // Output: Hello\nBye
```

### Output:
```
Hello
Bye
```

## Explanation

- `Notify` is a custom delegate type that matches any method returning `void` with no parameters.
- `SayHello` and `SayBye` are two methods matching the delegate signature.
- We assign `SayHello` to the delegate, then append `SayBye` using the `+=` operator.
- When calling `notify()`, both methods are invoked **in order**, producing the combined output.

## Notes

- Only delegates with `void` return type support true multicast behavior.
- If the delegate has a return type (e.g., `int`), only the return value of the **last method** in the invocation list will be returned.
- You can use `-=` to remove methods from the invocation list:
  ```csharp
  notify -= SayHello;
  ```

## When to Use

- Event systems
- Logging or notification chains
- Scenarios where a single trigger should result in multiple actions

---

Happy coding!

## Calling Delegates: `()` vs `.Invoke()`

Delegates in C# can be called using either the shorthand syntax `delegateName()` or the explicit method call `delegateName.Invoke()`.

Both are functionally equivalent:

```csharp
notify();           // Shorthand
notify.Invoke();    // Equivalent, more explicit
```

### When to Use `.Invoke()`?

- When you want to use **null-safe calls**:
  ```csharp
  Notify? notify = SomeCondition ? SayHello : null;
  notify?.Invoke(); // Won't throw if notify is null
  ```

- When writing **more readable or formal** code in certain contexts

Both styles are valid, and choosing between them is mostly a matter of preference or readability.
