## 🚀 مرحله ۱۲ — Deploy Docker Image از ACR به Azure App Service / Container Apps

### 🎯 هدف
بعد از اینکه Pipeline، Docker Image را Build و Push کرد، حالا می‌خواهیم این Image را روی محیط عملیاتی (Production / Test) اجرا کنیم.

---

### 🔹 گام ۱ — انتخاب نوع سرویس

می‌توانی بین دو گزینه انتخاب کنی:

1. **Azure App Service (Container)**  → برای استقرار سریع و ساده.
2. **Azure Container Apps / AKS**  → برای پروژه‌های بزرگ‌تر و نیاز به مقیاس‌پذیری بالا.

---

### 🔹 گام ۲ — تنظیم App Service برای Docker

1. وارد **Azure Portal → App Services → AzureDemoApi-vahidfaraji** شو.
2. به مسیر **Settings → Configuration → Container Settings** برو.
3. مقادیر زیر را تنظیم کن:
   - **Image Source:** Azure Container Registry
   - **Registry:** `azuredemoacr2873`
   - **Image and Tag:** `azuredemoapi:1.0.0` (یا هر Tag دلخواه)
   - **Authentication Type:** Managed Identity یا Admin User (مطابق با Service Connection)
4. تنظیمات را **Save** کن و App را **Restart** کن.

---

### 🔹 گام ۳ — تست نهایی

با مرورگر یا Postman آدرس زیر را تست کن:

```bash
http://<AppServiceName>.azurewebsites.net/api/hello
```

✅ اگر خروجی زیر را دیدی، Deploy با موفقیت انجام شده:

```json
{"message": "Hello from AzureDemoApi!"}
```

---

### ⚙️ نکات مهم

- اگر بخواهی **Deploy اتوماتیک** انجام شود، می‌توانی در Pipeline مرحله‌ای اضافه کنی تا بعد از Push، Image جدید به‌صورت خودکار روی App Service مستقر شود.
- این کار باعث می‌شود CI/CD کامل به‌صورت خودکار انجام شود (**Build → Push → Deploy**).

---

اگر آماده‌ای، در قدم بعدی می‌رویم سراغ **اضافه کردن مرحله‌ی Deploy اتوماتیک به Pipeline** تا هر Build جدید مستقیماً روی App Service مستقر شود. 🚀

