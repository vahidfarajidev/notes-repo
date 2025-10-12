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
3. از بالای صفحه، **URL** برنامه را کپی کن، مثلا:

```
https://azuredemoapi-vahidfaraji.azurewebsites.net
```

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

**«قدم بعدی»**: **اضافه کردن مانیتورینگ و Application Insights** برای مشاهده لاگ‌ها و سلامت اپلیکیشن.

