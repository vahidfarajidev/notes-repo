# مرحله ۲ — آماده‌سازی پروژه برای CI/CD

هدف این مرحله این است که مطمئن شویم پروژه آماده است تا روی سرور CI/CD (مثل Azure DevOps Pipeline) بدون نیاز به Visual Studio و به‌صورت خودکار Build و Test شود.

---

## 🔹 گام ۲.۱ — ایجاد Git Repository

1. اگر پروژه هنوز Git ندارد، در پوشه‌ی اصلی پروژه دستور زیر را اجرا کن:

   ```bash
   git init
   ```

2. فایل `.gitignore` مخصوص پروژه‌های .NET را اضافه کن (برای جلوگیری از Commit شدن فایل‌های غیرضروری):

   ```bash
   dotnet new gitignore
   ```

3. تغییرات اولیه را Stage و Commit کن:

   ```bash
   git add .
   git commit -m "Initial commit: HelloController API"
   ```

---

## 🔹 گام ۲.۲ — ساخت Branch اصلی

اگر شاخه‌ی اصلی هنوز وجود ندارد یا نامش چیز دیگری است، آن را به `main` تغییر بده:

```bash
git branch -M main
```

> 💡 در اکثر سرویس‌های CI/CD شاخه‌ی پیش‌فرض `main` است، نه `master`. بهتر است از همین ابتدا نام شاخه را تنظیم کنی.

---

## 🔹 گام ۲.۳ — تست Build بدون IDE

قبل از اینکه پروژه به Azure DevOps منتقل شود، باید مطمئن شویم که فقط با ابزار CLI (`dotnet`) به‌درستی Build و Test می‌شود.

```bash
dotnet restore
dotnet build
dotnet test
dotnet publish -c Release -o ./publish
```

✅ اگر همه‌ی مراحل بدون خطا اجرا شد، پروژه‌ی تو آماده‌ی CI/CD است.

---

### 💬 نکته‌ها

- `dotnet restore` → دانلود وابستگی‌های پروژه از NuGet  
- `dotnet build` → ساخت (Compile) پروژه  
- `dotnet test` → اجرای تست‌های واحد (Unit Tests)  
- `dotnet publish` → خروجی نهایی پروژه برای استقرار (Deploy)

> 🔸 نکته: گزینه‌ی `-c` یعنی Configuration (مثل Release یا Debug) و `-o` یعنی Output Directory، مسیر خروجی ساخت.

---

### 🎯 نتیجه نهایی

در پایان این مرحله:

- پروژه در Git قرار دارد.  
- شاخه‌ی اصلی (`main`) مشخص است.  
- پروژه بدون Visual Studio و فقط با دستورهای CLI قابل Build و Test است.  

بنابراین آماده‌ای تا در مرحله‌ی بعد (مرحله ۳) وارد **Azure DevOps** شده و فرآیند **CI/CD Pipeline** را بسازی 🚀

---

