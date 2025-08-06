# When to Create Domain Services—and When Not To

As a DDD practitioner, one of the most important decisions you face is where to put business logic:  
- Should it live inside a domain entity or value object?  
- Should it go into a domain service?  
- Or should it remain in the application layer?

This article aims to answer these questions by revisiting the *purpose* of Domain-Driven Design and examining the role domain services play in a clean, expressive model.

---

## Why Use Domain-Driven Design in the First Place?

Enterprise applications are inherently complex—not just because of technical concerns, but because they solve **problems developers don’t fully understand** at the outset.

The **real experts** are often domain stakeholders: finance officers, logistics managers, operations leads. Developers, no matter how skilled, don’t share that same intuitive knowledge of the domain. That’s why we talk to domain experts.

So, instead of coding based on guesswork, we design our software around the **ubiquitous language** and **mental model** of these domain experts. And that’s where the heart of DDD lies.

### The Core Principle

DDD encourages us to group behavior with data by placing **logic inside domain entities and value objects**—not out of dogma, but because that's where the **meaning** of that logic belongs.

When an `Invoice` entity decides whether it is `Overdue`, or a `Customer` object calculates its `LoyaltyTier`, that logic is rooted in the language and rules of the business domain. The code becomes readable not just by developers, but by those who **own the problem**.

> **“Place operations on the domain model types where they seem to belong most naturally, according to the meaning of the operation and the meaning of the model elements.”**  
> — *Eric Evans*, DDD Book

---

## But Then—What Are Domain Services?

Sometimes, however, logic doesn’t quite fit inside a single entity or value object. It requires **coordination** between multiple parts of the domain. That’s when domain services come in.

A **domain service** is a stateless object that expresses a significant domain operation or rule that **does not naturally belong** to a single entity.

> **“A SERVICE is an operation offered as an interface that stands alone in the model, without fitting naturally into an ENTITY or VALUE OBJECT.”**  
> — *Eric Evans*, DDD Book

### When to Use a Domain Service

Use a domain service when:
- The operation **involves multiple entities or value objects**
- The operation is **pure domain logic**
- The logic doesn’t fit naturally inside any one aggregate without violating encapsulation

#### Example

```csharp
public class TransferService
{
    public void TransferMoney(Account from, Account to, Money amount)
    {
        if (!from.CanWithdraw(amount))
            throw new InvalidOperationException("Insufficient funds.");

        from.Withdraw(amount);
        to.Deposit(amount);
    }
}
```

> **“Modelers must resist the temptation to treat domain services as procedural utilities.”**  
> — *Vaughn Vernon*, Implementing Domain-Driven Design

---

## When **Not** to Use a Domain Service (And What to Do Instead)

Despite being part of the DDD toolbox, **domain services are rarely needed**. Overusing them leads to an anemic domain model—a procedural codebase where behavior is detached from the data it belongs to.

> **“Avoid the anemic domain model. Behavior is what brings models to life. A model without behavior is like a body without a soul.”**  
> — *Vaughn Vernon*

> **“Every time you use a domain service, ask yourself if it’s because your model lacks an appropriate concept.”**  
> — *Jimmy Nilsson*

---

### ❌ Anti-Pattern: Using a Domain Service for Behavior That Belongs in the Entity

```csharp
public class OrderService
{
    public void Cancel(Order order)
    {
        if (order.Status != OrderStatus.Paid)
            throw new InvalidOperationException("Only paid orders can be cancelled.");

        order.Status = OrderStatus.Cancelled;
    }
}
```

✅ **Better: Put it in the Aggregate**

```csharp
public class Order
{
    public void Cancel()
    {
        if (Status != OrderStatus.Paid)
            throw new InvalidOperationException("Only paid orders can be cancelled.");

        Status = OrderStatus.Cancelled;
    }
}
```

---

### ❌ Anti-Pattern: Extracting Logic Just to “Re-use” It

```csharp
public class SalaryService
{
    public Money CalculateTax(Money grossSalary)
    {
        return grossSalary * 0.25;
    }
}
```

✅ **Better: Put it in a Value Object**

```csharp
public class Money
{
    public Money CalculateTax(double rate)
    {
        return new Money(this.Amount * rate);
    }
}
```

---

### ❌ Anti-Pattern: Procedural Logic Across Aggregates

```csharp
public class OrderShippingService
{
    public void ShipOrder(Order order, Inventory inventory)
    {
        if (!inventory.HasStock(order.ProductId))
            throw new InvalidOperationException("Out of stock.");

        inventory.DecreaseStock(order.ProductId);
        order.MarkAsShipped();
    }
}
```

✅ **Better: Let Aggregates Own Their Own Behavior**

```csharp
public class Inventory
{
    public void ReserveProduct(Guid productId) { ... }
}

public class Order
{
    public void Ship() { ... }
}
```

---

## ✅ When You **Should** Use a Domain Service

> **“If the logic can live in the domain entity or value object, it should.”**

Use a domain service **only when** the logic:
- Spans multiple aggregates
- Has no clear home inside an object
- Expresses a domain concern (not just orchestration)

---

### ✅ Example: Money Transfer

```csharp
public class MoneyTransferService
{
    public void Transfer(Account from, Account to, Money amount)
    {
        if (!from.CanWithdraw(amount))
            throw new InvalidOperationException("Insufficient funds.");

        from.Withdraw(amount);
        to.Deposit(amount);
    }
}
```

---

### ✅ Example: Authorization

```csharp
public class AuthorizationService
{
    public bool CanViewReport(User user, Report report)
    {
        return user.Roles.Contains("Manager") || report.OwnerId == user.Id;
    }
}
```

---

### ✅ Example: Policy Evaluation

```csharp
public class LoanApprovalService
{
    public bool CanApproveLoan(Customer customer, LoanApplication application)
    {
        return customer.CreditScore > 700 &&
               application.Amount <= customer.MonthlyIncome * 10;
    }
}
```

> **“When you have a domain service that coordinates multiple aggregates, make sure you're not just hiding transaction scripts inside a fancy class.”**  
> — *Greg Young*

---

## Summary

| ❌ Don’t Use If...                              | ✅ Use If...                                         |
|------------------------------------------------|------------------------------------------------------|
| The logic belongs to a single entity or value object | The logic involves multiple aggregates/entities     |
| It’s just coordination logic for a use case     | It expresses a domain rule that's not entity-specific |
| You’re tempted to "share logic" artificially   | It maps directly to a business concept               |
| The behavior fits naturally in the object model | A domain expert would describe it as a service       |

---

> **“Try to place the behavior on VALUE OBJECTS or ENTITIES. Only when the behavior doesn't conceptually belong to any object should you consider adding a domain service.”**  
> — *Eric Evans*

---
**Behrang Mohseni** | Auguest, 2025 | behrangmohseni@live.com