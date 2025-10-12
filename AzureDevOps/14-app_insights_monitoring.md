# 🚀 فعال‌سازی مانیتورینگ و Application Insights

حالا بریم فقط **یک قدم بعدی** — یعنی **اضافه کردن مانیتورینگ و Application Insights** برای مشاهده‌ی لاگ‌ها و سلامت اپلیکیشن.

---

## 🔹 مرحله ۸ — فعال‌سازی Application Insights

### 🎯 هدف این قدم

جمع‌آوری Telemetry، لاگ‌ها، خطاها و Performance اپلیکیشن تا بتونی سلامت و رفتار API رو در محیط عملیاتی بررسی کنی.

---

### گام ۸.۱ — ایجاد Application Insights در Azure

1. وارد [Azure Portal](https://portal.azure.com) شو
2. از منوی بالا یا سمت چپ روی **Create a resource** → جستجو کن **Application Insights**
3. Create
4. پر کن بخش‌ها:

   * **Name:** `AzureDemoApi-Insights`
   * **Resource Group:** همون `AzureDemo-RG`
   * **Region:** همون Region که App Service در آن ساخته شد
   * **Resource Mode:** Classic یا Workspace-based (می‌تونی Classic انتخاب کنی)
5. Create ✅

---

### گام ۸.۲ — اتصال App Service به Application Insights

1. به **App Service → AzureDemoApi-vahidfaraji** برو
2. از منوی سمت چپ **Settings → Configuration → Application Settings**
3. بررسی کن که **Application Insights** فعال شده باشه
4. اگر لازم بود، **Instrumentation Key** یا **Connection String** که در Application Insights ایجاد شده، در App Settings اضافه کن
5. App Service را Restart کن تا تغییرات اعمال شود

---

### گام ۸.۳ — بررسی Telemetry

* بعد از چند دقیقه، وارد **Application Insights → AzureDemoApi-Insights → Live Metrics** شو
* باید درخواست‌های GET به `/api/hello` و Performance، Exceptions و Request Logs را ببینی

---

✅ حالا اپلیکیشن تو هم Deploy شده و هم مانیتورینگ فعال داره و می‌توانی سلامت و عملکردش را آنلاین رصد کنی.

---

**قدم بعدی**: **تنظیم Alerts و Notification برای خطاها و مشکلات احتمالی**.

