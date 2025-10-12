# 🚀 تست نهایی Web API روی Azure

حالا بریم فقط **یک قدم بعدی** — یعنی **تست نهایی و بررسی اجرای Web API روی Azure**.

---

## 🔹 مرحله ۷ — تست نهایی Web API روی Azure

### 🎯 هدف این قدم

اطمینان پیدا کنیم که **Deploy موفقیت‌آمیز** بوده و API آنلاین قابل دسترسی است.

---

### گام ۷.۱ — پیدا کردن URL App Service

1. وارد [Azure Portal](https://portal.azure.com) شو.
2. به **App Services** → **AzureDemoApi-vahidfaraji** برو.
3. از بالای صفحه، **Default Domain** برنامه را در نظر بگیر و مسیر `api/hello` را به آن اضافه کن، مثلا:

```
https://azuredemoapi-vahidfaraji-bcf2b4bef7fpdzdj.westeurope-01.azurewebsites.net/api/hello
```

> ⚠️ نکته: این Default Domain واقعی Azure است که به صورت خودکار ساخته می‌شه.
>
> نکته‌هاش:
> 1. **طولانی و شامل شناسه تصادفی** است. این شناسه برای اطمینان از یکتا بودن نام در سطح جهانی اضافه می‌شه.
> 2. هنوز **صفحه‌ی پیش‌فرض App Service** نمایش داده می‌شه، نه خروجی API، چون احتمالاً:
>    * فایل‌های Publish روی App Service درست Deploy نشده‌اند.
>    * یا **مسیری که در Browser می‌بینی** (`/`) هیچ Endpoint خاصی ندارد.
> 3. اگر هنوز پاسخی نمی‌بینی:
>    * Pipeline را بررسی کن که **Deploy موفق بوده** (مرحله Deploy سبز باشد).
>    * App Service → **Deployment Center** را چک کن تا ببینی فایل‌ها واقعاً آپلود شده‌اند.
>    * اطمینان حاصل کن **Startup / launchSettings** پروژه درست تنظیم شده و API روی مسیر `/api/hello` در دسترس است.


---

### گام ۷.۲ — تست Endpoint

1. در مرورگر یا با ابزارهایی مثل **Postman** برو به:

```
https://azuredemoapi-vahidfaraji.azurewebsites.net/api/hello
```

2. باید پاسخ `"Hello from AzureDemoApi!"` را ببینی ✅

---

اگر پاسخ درست بود، یعنی **CI/CD کامل و موفق بوده** و Web API تو به صورت خودکار از Azure DevOps به Azure App Service Deploy شده.

---

**قدم بعدی**: **اضافه کردن مانیتورینگ و Application Insights** برای مشاهده لاگ‌ها و سلامت اپلیکیشن.

