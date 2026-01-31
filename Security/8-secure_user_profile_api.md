# امن کردن مسیرهای پروفایل با /me در ASP.NET Core

این مقاله نحوه امن کردن مسیرهای نمایش و ویرایش پروفایل کاربران در یک وب‌سایت با ASP.NET Core را توضیح می‌دهد، به‌ویژه جلوگیری از **IDOR (Insecure Direct Object Reference)**.

---

## مشکل IDOR

فرض کنید یک API داریم:
```
GET /users/21/profile
```
- 21 شناسه کاربر است.
- اگر کاربر URL را تغییر دهد، مثلا `/users/22/profile`، پروفایل دیگران نمایش داده می‌شود.

این مشکل، **IDOR** است.

---

## راه حل: مسیر `/me`

### ایده کلی
- کاربر خودش **ID نفرستد**.
- سرور از **توکن JWT** یا **Session** ID کاربر را بگیرد.
- مسیر API ساده شود:
```
GET /me/profile
PUT /me/profile
```
- سرور خودش کاربر جاری را تشخیص می‌دهد.

### مزایا
1. کاربر نمی‌تواند ID دیگران را حدس بزند.
2. مسیرها ساده و واضح می‌شوند.
3. جلوگیری کامل از IDOR بدون کنترلرهای اضافی.

---

## مثال عملی در ASP.NET Core

### 1️⃣ مدل کاربر
```csharp
public class User
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
}
```

### 2️⃣ DbContext
```csharp
using Microsoft.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    public DbSet<User> Users { get; set; }

    public AppDbContext(DbContextOptions<AppDbContext> options)
        : base(options)
    {
    }
}
```

### 3️⃣ Controller با مسیر `/me/profile`
```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using System.Security.Claims;

[ApiController]
[Route("[controller]")]
public class MeController : ControllerBase
{
    private readonly AppDbContext _context;

    public MeController(AppDbContext context)
    {
        _context = context;
    }

    [HttpGet("profile")]
    [Authorize]
    public IActionResult GetProfile()
    {
        var currentUserIdString = User.FindFirstValue(ClaimTypes.NameIdentifier);
        if (currentUserIdString == null)
            return Unauthorized();

        int currentUserId = int.Parse(currentUserIdString);

        var user = _context.Users.FirstOrDefault(u => u.Id == currentUserId);
        if (user == null)
            return NotFound();

        return Ok(new { user.Id, user.Username, user.Email });
    }

    [HttpPut("profile")]
    [Authorize]
    public IActionResult UpdateProfile([FromBody] User model)
    {
        var currentUserIdString = User.FindFirstValue(ClaimTypes.NameIdentifier);
        int currentUserId = int.Parse(currentUserIdString);

        var user = _context.Users.FirstOrDefault(u => u.Id == currentUserId);
        if (user == null)
            return NotFound();

        user.Email = model.Email;
        user.Username = model.Username;
        _context.SaveChanges();

        return Ok(user);
    }
}
```

---

## نحوه تشخیص کاربر بدون ID در URL

### 1️⃣ با Session

#### لاگین کاربر
```csharp
HttpContext.Session.SetInt32("UserId", user.Id);
```
- سرور یک SessionID تصادفی می‌سازد، مثلا `abc123`.
- SessionID در **Cookie** مرورگر ذخیره می‌شود:
```
Set-Cookie: .AspNetCore.Session=abc123; HttpOnly; Secure
```

#### درخواست بعدی مرورگر
```
GET /me/profile
Cookie: .AspNetCore.Session=abc123
```

#### گرفتن UserId در سرور
```csharp
var userId = HttpContext.Session.GetInt32("UserId");
```
- فریم‌ورک خودش SessionID را از Cookie می‌خواند.
- SessionStore داخلی را نگاه می‌کند:
```
SessionStore = { "abc123": { "UserId": 21 } }
```
- مقدار 21 برگردانده می‌شود و برنامه فقط با `userId` کار می‌کند.

نکته: HttpContext.Session یک wrapper است روی SessionStore که با SessionID مطابقت داده شده، وقتی می‌گویی GetInt32("UserId")، فریم‌ورک خودش می‌رود به SessionStore مرتبط با SessionID و مقدار "UserId" را برمی‌گرداند. در واقع تو هیچوقت SessionID را نمی‌بینی یا دستکاری نمی‌کنی، فقط می‌پرسی «برای این Session چه مقدار UserId است؟»

> نکته: برنامه‌نویس هیچوقت با SessionID کار نمی‌کند، فقط داده‌ها را از Session می‌گیرد.

### 2️⃣ با JWT

#### صادر کردن JWT هنگام لاگین
```csharp
var claims = new[] { new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()) };
var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("secret_key"));
var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
var token = new JwtSecurityToken(
    claims: claims,
    expires: DateTime.Now.AddHours(1),
    signingCredentials: creds);
var jwt = new JwtSecurityTokenHandler().WriteToken(token);
```
- JWT شامل UserId است.

#### درخواست به `/me/profile`
```
GET /me/profile
Authorization: Bearer <JWT>
```
- سرور توکن را decode می‌کند:
```csharp
var currentUserId = User.FindFirstValue(ClaimTypes.NameIdentifier);
```
- همان UserId استخراج می‌شود.

---

## جمع‌بندی جریان امن `/me/profile`

1. کاربر لاگین می‌کند → Session یا JWT ایجاد می‌شود.
2. مرورگر درخواست `/me/profile` می‌فرستد و SessionID یا JWT را ارسال می‌کند.
3. سرور SessionID را می‌خواند یا JWT را decode می‌کند → UserId استخراج می‌شود.
4. دیتابیس فقط برای همین UserId فیلتر می‌شود.
5. پاسخ برمی‌گردد.

---

## نکات مهم و توصیه‌ها

- مسیر `/me` یک convention رایج است، نه یک قابلیت سیستمی.
- از مسیر `/users/21/profile` برای کاربر جاری استفاده نکنید، چون امکان IDOR وجود دارد.
- فریم‌ورک ASP.NET Core مدیریت SessionID را انجام می‌دهد و برنامه‌نویس فقط داده‌ها را می‌گیرد.
- مسیرهای `/account`, `/self`, `/profile` هم می‌توانند استفاده شوند ولی `/me` رایج‌ترین و استانداردترین است.

---

### منابع و استانداردها
- [OWASP IDOR](https://owasp.org/www-community/attacks/Insecure_Direct_Object_References)
- [JWT](https://jwt.io/)
- [ASP.NET Core Session Documentation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/app-state?view=aspnetcore-7.0)
- استانداردهای REST API در سرویس‌های بزرگ مثل GitHub، Google و Facebook که مسیر `/me` را استفاده می‌کنند.

-  توجه: در خصوص دریافت اطلاعات کاربر این کار امکان پذیر است اما در خصوص سناریوهایی مثلا دریافت جزئیات یک سفارش، بایستی از سمت مروگر، شناسه ای یا Id فرستاده شود. در این سناریوها نکته این است باید موضوع IDOR رعایت شود یعنی باید بررسی گردد آن کاربر واقعا به آن منبع مورد نظر یعنی در اینجا سفارش، دسترسی دارد یا خیر. یا به عبارنی این سفارش مربوط به این کاربر است یا خیر. مثلا ممکن است Id دیگری به سمت سرور فرستاده شود که اصلا متعلق به کاربر لاگین شده نباشد.
