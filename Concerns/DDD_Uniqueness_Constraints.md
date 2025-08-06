# Designing Around Uniqueness Constraints in Domain-Driven Design

As DDD practioner, I often work with teams trying to build expressive domain models using Domain-Driven Design (DDD). One common challenge in real-world systems is handling rules that require **global knowledge** — like enforcing the uniqueness of a user’s email or username.

When modeling such rules, we usually want three things:
- **A complete domain model**: all business logic lives in the domain layer.
- **A clean model**: no out-of-process dependencies like repositories inside entities.
- **An efficient system**: performance that scales with real data.

But enforcing uniqueness across many entities — such as checking if a username is already taken — puts these goals at odds.

Let’s break this down through an example.

## Example: Creating a Unique Username

Consider a user registration process. The business rules are:
1. The username must be unique.
2. It must not include banned terms.

Here’s a first attempt where we enforce all rules inside the domain entity:

```csharp
public class User : Entity
{
    public string Username { get; private set; }

    public Result SetUsername(string newUsername, List<User> allUsers)
    {
        if (allUsers.Any(u => u.Username == newUsername))
            return Result.Failed("Username is already taken");

        if (IsBanned(newUsername))
            return Result.Failed("Username contains a banned word");

        Username = newUsername;
        return Result.Succeeded();
    }

    private bool IsBanned(string username)
    {
        return new[] { "admin", "root", "system" }
            .Any(b => username.Contains(b, StringComparison.OrdinalIgnoreCase));
    }
}
```

### What’s Wrong?

This model is **complete** and **pure**. All the logic is in one place and the entity is not coupled to infrastructure.

But it’s **not scalable**. We can’t load millions of users into memory just to check one username.

---

## Option 2: Injecting a Repository into the Entity

One workaround is to inject a repository into the domain entity and delegate the uniqueness check to it:

```csharp
public class User : Entity
{
    public string Username { get; private set; }

    public Result SetUsername(string newUsername, IUserRepository userRepository)
    {
        if (userRepository.ExistsByUsername(newUsername))
            return Result.Failed("Username is already taken");

        if (IsBanned(newUsername))
            return Result.Failed("Username contains a banned word");

        Username = newUsername;
        return Result.Succeeded();
    }

    private bool IsBanned(string username)
    {
        return new[] { "admin", "root", "system" }
            .Any(b => username.Contains(b, StringComparison.OrdinalIgnoreCase));
    }
}
```

### Pros and Cons

- ✅ Avoids loading all users.
- ✅ Keeps logic in the entity.
- ❌ Introduces an out-of-process dependency into the domain — breaking purity and making testing harder.

---

## Option 3: Delegating Uniqueness Check to the Application Layer

Instead of compromising purity or performance, we can move part of the validation to the application layer:

```csharp
public class UserService
{
    private readonly IUserRepository _userRepository;

    public Result RegisterUser(string username)
    {
        if (_userRepository.ExistsByUsername(username))
            return Result.Failed("Username is already taken");

        var user = new User();
        var result = user.SetUsername(username);

        if (!result.Succeeded())
        {
            return Result.Failed(result.Message);
        }

        _userRepository.Save(user);
        return Result.Succeeded();
    }
}
```

And the entity becomes more pure:

```csharp
public class User : Entity
{
    public string Username { get; private set; }

    public void SetUsername(string username)
    {
        Username = username;
    }

    private bool IsBanned(string username)
    {
        return new[] { "admin", "root", "system" }
            .Any(b => username.Contains(b, StringComparison.OrdinalIgnoreCase));
    }
}
```

### Trade-Offs

- ✅ High performance — only the necessary query is executed.
- ✅ High testability — the domain is clean and decoupled.
- ❌ Logic is fragmented — some in the domain, some in the application service.

---

## Choosing the Right Balance

Here’s how the options stack up:

| Approach                           | Domain Purity | Logic Completeness | Performance |
|-----------------------------------|----------------|---------------------|-------------|
| Entity holds all users            | ✅             | ✅                  | ❌          |
| Entity with repository dependency | ❌             | ✅                  | ✅          |
| Application layer coordination    | ✅             | ❌ (fragmented)     | ✅          |

Personally, I lean toward the **third approach** in most real-world systems. It favors simplicity, testability, and performance, even though it fragments logic.

If needed, you can extract the validation into a **domain service** to keep logic in the domain layer — without violating purity. Just be cautious not to move all logic out of your entities by default.

---

## Final Thoughts

Domain-Driven Design encourages us to model business rules clearly and close to where they belong. But not every rule fits neatly inside an entity — especially when it relies on global state or distributed data.

When you find yourself passing too much data into an entity just to validate something, it’s a sign that **some logic belongs elsewhere**.

Keep your domain clean, your boundaries intentional, and your architecture practical.

---
**Behrang Mohseni** | Auguest, 2025 | behrangmohseni@live.com