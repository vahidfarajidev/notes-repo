# نصب Azure CLI در ویندوز و آماده‌سازی محیط برای Docker & ACR

این راهنما شامل مراحل نصب **Winget**، **Azure CLI** و آماده‌سازی محیط برای کار با **Docker** و **Azure Container Registry (ACR)** است.

---

## ۱️⃣ بررسی وجود Winget

Winget همان **Windows Package Manager** است که ابزار خط فرمانی برای نصب نرم‌افزارهاست.

در Developer PowerShell یا PowerShell عادی دستور زیر را اجرا کن:

```powershell
winget --version
```

- اگر نسخه‌ای نمایش داده شد → Winget نصب است و می‌توانی ادامه دهی.  
- اگر پیام خطا داد که `winget` شناخته نشده، یعنی هنوز نصب نشده و باید ادامه مراحل را انجام دهیم.

---

## ۲️⃣ نصب Winget (در صورت عدم وجود)

اگر Winget نصب نیست، مراحل زیر را انجام بده:

1. به صفحه رسمی GitHub Winget برو:  
🔗 [Winget Releases](https://github.com/microsoft/winget-cli/releases)

2. نسخه مناسب برای **Windows 10/11، 64 بیتی** را پیدا کن.  
   - فایل دانلودی: `Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle`  
   - حتماً فایل MSIXBundle را انتخاب کن، نه EXE.

3. نصب فایل MSIXBundle از طریق PowerShell:  
   - PowerShell را **به صورت Administrator باز کن**  
   - دستور زیر را اجرا کن (مسیر فایل را به مسیر واقعی که دانلود کردی تغییر بده):

```powershell
Add-AppxPackage -Path "F:\SoftwareNew\0_ProgrammingTools\Microsoft.DesktopAppInstaller_8wekyb3d8bbwe.msixbundle"
```

4. بعد از نصب، **تمام پنجره‌های PowerShell یا Developer PowerShell را ببند و دوباره باز کن** تا مسیر Winget شناسایی شود.

---

## ۳️⃣ نصب Azure CLI با Winget

پس از نصب Winget، Azure CLI را نصب کن:

```powershell
winget install -e --id Microsoft.AzureCLI
```

- پس از اتمام نصب، Developer PowerShell را باز کن و بررسی کن Azure CLI نصب شده است:

```powershell
az --version
```

خروجی باید چیزی شبیه این باشد:

```
azure-cli                         2.77.0 *
core                              2.77.0 *
...
```

✅ اگر نسخه نمایش داده شد، Azure CLI آماده استفاده است.

---

## ۴️⃣ ورود به حساب Azure

برای ورود به حساب Azure:

```powershell
az login
```

- مرورگر باز می‌شود و از تو می‌خواهد وارد حساب Azure شوی.  
- پس از ورود موفق، CLI آماده کار با Subscription و منابع Azure است.  

💡 نکته‌ها:

- اگر چند Subscription داری، CLI از تو می‌خواهد یکی را انتخاب کنی.  
- بعد از لاگین، تمام دستورهای Azure CLI مانند ساخت Resource Group، ACR و Push Docker image قابل استفاده هستند.

---

### ✨ جمع‌بندی

| مرحله | دستور / اقدام | توضیح |
|-------|----------------|-------|
| بررسی Winget | `winget --version` | بررسی اینکه Windows Package Manager نصب است |
| نصب Winget | دانلود MSIXBundle و `Add-AppxPackage` | نصب App Installer و Winget در ویندوز |
| نصب Azure CLI | `winget install -e --id Microsoft.AzureCLI` | نصب ابزار رسمی مدیریت Azure |
| تست نصب | `az --version` | بررسی صحت نصب Azure CLI |
| ورود به Azure | `az login` | اتصال به حساب Azure و انتخاب Subscription |

---

بعد از این مرحله، محیط آماده است تا **Docker Image لوکال را به Azure Container Registry (ACR) Push کنی**.

