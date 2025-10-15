### 💡 Azure CLI چیست؟

**Azure CLI (Command-Line Interface)** یک ابزار خط فرمان رسمی از مایکروسافت است که برای مدیریت سرویس‌های Azure استفاده می‌شود. با آن می‌توانی همهٔ کارهایی را که در پورتال Azure انجام می‌دهی (مثل ساخت Resource Group، ایجاد Container Registry، استقرار App Service و...) از طریق ترمینال یا اسکریپت انجام دهی.

---

### ⚙️ نصب Azure CLI

اگر هنوز Azure CLI را نصب نکردی، می‌توانی از لینک رسمی مایکروسافت آن را نصب کنی:

🔗 [نصب Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli)

برای سیستم‌عامل‌ها:

- **Windows:**
  ```powershell
  winget install -e --id Microsoft.AzureCLI
  ```
- **macOS:**
  ```bash
  brew update && brew install azure-cli
  ```
- **Linux (Debian/Ubuntu):**
  ```bash
  curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
  ```

---

### 🚀 نحوه ورود به حساب کاربری (Login)

بعد از نصب، باید به حساب Azure خود وارد شوی:

```bash
az login
```

پس از اجرای دستور، مرورگر باز می‌شود تا وارد حساب مایکروسافت خود شوی. بعد از ورود، CLI توکن احراز هویت دریافت می‌کند و آمادهٔ استفاده می‌شود.

---

### 🔍 تست اتصال

برای اطمینان از اتصال درست، می‌توانی دستور زیر را بزن:

```bash
az account show --output table
```

اگر نام حساب و Subscription را نشان داد، یعنی لاگین موفقیت‌آمیز بوده و می‌توانی با CLI سرویس‌های Azure را مدیریت کنی.

---

### ✨ جمع‌بندی

| مفهوم | توضیح |
|--------|--------|
| **Azure CLI** | ابزار خط فرمان برای کار با سرویس‌های Azure |
| **مزیت اصلی** | انجام کارها به‌صورت خودکار و قابل اسکریپت‌نویسی |
| **دستور شروع** | `az login` برای ورود به حساب |
| **محل اجرا** | PowerShell، CMD یا Terminal (در macOS/Linux) |

---

اگر بخوای، می‌تونم برات مثال‌هایی از رایج‌ترین دستورات CLI برای کار با ACR یا Resource Group هم اضافه کنم تا در ادامه‌ی پروژه‌ات استفاده کنی.

