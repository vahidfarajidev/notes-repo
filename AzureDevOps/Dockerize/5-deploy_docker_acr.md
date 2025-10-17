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

قدم بعدی: **اضافه کردن مرحله‌ی Deploy اتوماتیک به Pipeline** تا هر Build جدید مستقیماً روی App Service مستقر شود. 🚀

---


## 🧩 واقعیت دربارهٔ چیزی که الان داری

### ✅ در واقع Pipeline الان **Image رو Push می‌کنه**

یعنی وقتی اجرا می‌شه:

1. کد Build می‌شه
2. Docker Image ساخته می‌شه
3. اون Image به Azure Container Registry (ACR) Push می‌شه
   (مثلاً `azuredemoacr2873.azurecr.io/azuredemoapi:1.0.0`)

📦 تا اینجا کار عالیه — یعنی بخش **CI (Continuous Integration)** کاملاً فعاله و سالم کار می‌کنه.

---

### ❌ ولی نه، هنوز **Image به App Service Deploy نمی‌کنه**

در واقع App Service فقط داره به **آخرین Image در ACR** اشاره می‌کنه — یعنی یک لینک ثابت داره به مثلاً:

```
azuredemoacr2873.azurecr.io/azuredemoapi:latest
```

و وقتی تو دستی App Service رو Restart کردی،
رفت آخرین نسخهٔ همون Image (tag آخرین push) رو گرفت و بالا آورد.

📌 且 یعنی Deploy واقعی از طریق Pipeline اتفاق نفتاد.
App Service فقط چون به اون Image وصل بود، بعد از Restart خودش Pull کرد.

---

### 🔍 یعنی چی از نظر فنی؟

در حال حاضر:

* Pipeline فقط تا مرحلهٔ “Push به ACR” جلو می‌ره ✅
* App Service خودش Deploy نمی‌کنه 🚫
* اگه tag جدیدی بزنی (`:1.0.1` مثلاً)، App Service خودش متوجه نمی‌شه مگر بری تنظیمش کنی یا Restart بزنی 🔁

---

### ✅ در CD واقعی چه می‌شه؟

در CD واقعی، آخرین stage در pipeline مثلاً اینه:

```yaml
- stage: Deploy
  jobs:
    - job: DeployToAppService
      steps:
        - task: AzureWebAppContainer@2
          inputs:
            azureSubscription: 'MyServiceConnection'
            appName: 'AzureDemoApi-vahidfaraji'
            imageName: 'azuredemoacr2873.azurecr.io/azuredemoapi:$(Build.BuildId)'
```

🟢 یعنی بعد از Push، همون Pipeline فوراً می‌ره App Service رو آپدیت می‌کنه تا از Image جدید استفاده کنه — بدون نیاز به ریستارت یا کار دستی.

---

### 🧩 خلاصه:

| کار                   | انجام می‌شه؟ | چطور        |
| --------------------- | ------------ | ----------- |
| Build Docker Image    | ✅            | در Pipeline |
| Push به ACR           | ✅            | در Pipeline |
| Deploy به App Service | ⚠️ خیر       | فع          |


