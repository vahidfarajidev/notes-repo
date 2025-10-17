## 🚀 افزودن مرحله‌ی Deploy خودکار به Azure App Service در Pipeline

---

### 🔹 مرحله ۱۳ — افزودن مرحله Deploy به App Service (بعد از Push به ACR)

در حال حاضر، Pipeline شما تا مرحله‌ی **Push Docker Image به Azure Container Registry (ACR)** پیش می‌رود.
در این مرحله، هدف ما این است که **Image جدید به‌صورت خودکار روی Azure App Service مستقر شود**.

---

### ✳️ گام ۱ — اضافه کردن Task جدید به YAML

فایل `azure-pipelines.yml` را باز کرده و این بخش را **بعد از مرحله‌ی Push** اضافه کنید:

```yaml
- task: AzureWebAppContainer@2
  displayName: Deploy to Azure App Service (Docker)
  inputs:
    azureSubscription: $(azureResourceManagerServiceConnection)   # همان Service Connection تعریف‌شده
    appName: $(appServiceName)         # نام واقعی App Service شما
    containers: '$(containerRegistry)/$(imageRepository):$(tag)' # تگ فعلاً ثابت در نظر گرفته شده است
```

---

### 🔍 توضیحات:

* **AzureWebAppContainer@1** مسئولیت دارد تا Image مشخص‌شده را از ACR گرفته و در App Service مستقر کند.
* `containers` مسیر کامل Image در ACR است →  
  `azuredemoacr2873.azurecr.io/azuredemoapi:$(tag)`
* `$(tag)` همان مقداری است که در مرحله Build/Push استفاده شده، پس نسخه دقیق Image مستقر خواهد شد.

---

### ✳️ گام ۲ — Commit و Push تغییرات

```powershell
git add azure-pipelines.yml
git commit -m "Add deploy to App Service step"
git push origin main
```

اکنون با اجرای Pipeline:

1. Image ساخته می‌شود ✅
2. در ACR ذخیره می‌شود ✅
3. همان Image روی Azure App Service مستقر می‌شود ✅

---

💡 آیا مایل هستید در قدم بعدی مرحله‌ای برای **تنظیم Environment Variables و Secrets از Azure Key Vault** اضافه کنیم تا بخش امنیتی CI/CD شما کامل‌تر شود؟

