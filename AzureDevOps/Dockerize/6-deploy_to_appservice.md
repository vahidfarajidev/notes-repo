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

💡 قدم بعدی: مرحله‌ای برای **تنظیم Environment Variables و Secrets از Azure Key Vault** اضافه کنیم تا بخش امنیتی CI/CD شما کامل‌تر شود.

---

# اهمیت مرحله Deploy در YAML حتی با تگ ثابت

حتی وقتی از **تگ ثابت** استفاده می‌کنیم، مرحله‌ی **Deploy** در pipeline حیاتی است. بدون این مرحله، CI کامل ولی CD ناقص خواهد بود.

---

## 🔹 چرا Deploy هنوز لازم است؟

1. **Push به ACR ≠ Deploy به App Service**
   * Pipeline فقط Image را به **ACR** می‌فرستد.
   * App Service به صورت خودکار آخرین Image در ACR را نمی‌گیرد مگر Manual Pull یا Restart انجام شود.

2. **بدون Deploy در pipeline**
   * App Service با هر Push تغییر نمی‌کند.
   * نیاز به ورود به Portal یا Restart دستی برای Pull Image جدید است.

3. **حتی با تگ ثابت (`1.0.0`)**
   * Push جدید فقط Image قبلی را در ACR overwrite می‌کند.
   * App Service تا اجرای Deploy، همچنان از نسخه قدیمی استفاده می‌کند.
   * مرحله Deploy App Service را به آخرین Image آپدیت می‌کند.

---

## 🔹 جریان دقیق

1. App Service **container image با تگ مشخص** را یک بار در زمان Start یا Restart Pull می‌کند.
2. Docker / App Service تگ ثابت را **immutable** در نظر می‌گیرد.
3. اگر Image در registry آپدیت شود اما App Service ریستارت نشود، همچنان نسخه قدیمی اجرا می‌شود.

---

## 🔹 راهکارها برای Deploy خودکار

1. اضافه کردن مرحله Deploy در pipeline (`AzureWebAppContainer@2`) → Pull و ریستارت اتوماتیک انجام می‌شود.
2. استفاده از تگ پویا (`1.0.$(Build.BuildId)`) → App Service با تگ جدید خودش Pull می‌کند.

---

## 🔹 جمع‌بندی کوتاه

| حالت                         | نتیجه                                                       |
| ---------------------------- | ----------------------------------------------------------- |
| تگ ثابت و Push بدون Deploy   | App Service نسخه قدیمی را اجرا می‌کند → نیاز به Restart    |
| تگ ثابت و Deploy در pipeline | App Service نسخه جدید را Pull و اجرا می‌کند ✅             |
| تگ پویا و Deploy             | App Service خودکار نسخه جدید را Pull می‌کند ✅             |

**نتیجه:** حتی با تگ ثابت، **ری‌استارت یا اجرای Deploy در pipeline ضروری است** تا آخرین Image در App Service اجرا شود.



