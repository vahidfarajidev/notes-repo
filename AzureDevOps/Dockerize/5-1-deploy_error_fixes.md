# جمع‌بندی خطاها و راه‌حل‌ها برای Azure App Service

### ✅ لیست خطاها و راه‌حل‌ها

**۱. Settings → Configuration → Container Settings نبود**

* **دلیل:** App Service جدید در Portal جدید، مسیر قدیمی Configuration → Container Settings وجود نداشت
* **راه‌حل:** از **بخش Search در Azure Portal**، App Service خود را پیدا کردیم و باز کردیم، سپس وارد بخش **Container Settings** شدیم. کانتینر اصلی (`main`) را انتخاب کردیم و پارامترهای زیر را ست کردیم:

  * **Image:** `azuredemoapi:1.0.0`
  * **Registry:** `azuredemoacr2873`
  * **Port:** `80`
  * **Identity:** `System Assigned`
    بعد Save و Restart App Service شد تا تغییرات اعمال شود

**۲. خطای Application Error روی `/api/hello`**

* **دلیل:** از روی **Log Stream** متوجه شدیم که پورت داخلی کانتینر درست ست نشده و ASP.NET Core روی پورت تصادفی (مثلاً 8181) اجرا می‌شود
* **راه‌حل:** Dockerfile را اصلاح کردیم:

  * اضافه کردن `EXPOSE 80`
  * اضافه کردن `ENV ASPNETCORE_URLS=http://+:80`
    → این باعث شد ASP.NET Core حتماً روی پورت 80 گوش دهد و Azure App Service بتواند ترافیک را به کانتینر هدایت کند
* **نتیجه:** بعد از اصلاح Dockerfile و ریستارت App Service، مسیر `/api/hello` بدون خطا کار کرد

**۳. خطای 403 یا App Service Stopped**

* **دلیل:** App Service Free F1 محدودیت instance و منابع داشت
* **راه‌حل:** Plan را به **B1** ارتقا دادیم تا حداقل 1 instance فعال باشد

**۴. دسترسی ACR (Pull) خطا می‌دهد / Role Assignment**

* **دلیل:** کانتینر اپلیکیشن بالا نمی‌آمد زیرا App Service دسترسی لازم برای Pull کردن Image از **Azure Container Registry (ACR)** را نداشت. بدون این دسترسی، Authentication برای گرفتن Docker Image از ACR انجام نمی‌شد → کانتینر اجرا نمی‌شد و Application Error می‌داد.

* **چرا این دسترسی لازم است؟**

  * App Service برای اجرا کردن کانتینر، نیاز دارد Docker Image را از ACR Pull کند
  * Managed Identity یا Service Principal باید نقش **AcrPull** داشته باشد تا اجازه خواندن Image را داشته باشد
  * بدون این دسترسی، App Service نمی‌تواند Image را دانلود کند → اپلیکیشن بالا نمی‌آید

* **راه‌حل ایجاد دسترسی:**

  1. در **Azure Container Registry → Access Control (IAM)**، روی **Add role assignment** کلیک کردیم
  2. Role را **AcrPull** انتخاب کردیم (این نقش فقط اجازه Pull کردن Image را می‌دهد، بدون دسترسی‌های مدیریتی)
  3. Member را **System Assigned Identity** App Service خودمان انتخاب کردیم
  4. ذخیره شد

* **نتیجه:** بعد از دادن این دسترسی، App Service توانست Docker Image را از ACR Pull کند و اپلیکیشن بالا آمد ✅

**۵. کانتینر اجرا می‌شود ولی پورت اشتباه (8080 / 8181)**

* **دلیل:** Dockerfile پورت صریحاً مشخص نشده بود
* **راه‌حل:** اصلاح Dockerfile شامل:

  * `EXPOSE 80`
  * `ENV ASPNETCORE_URLS=http://+:80`
    → ASP.NET Core همیشه روی پورت 80 اجرا شود

**۶. Restart مکرر کانتینر بعد از تنظیم PORT**

* **دلیل:** ASP.NET Core داخل کانتینر پورت را override می‌کرد
* **راه‌حل:** با Dockerfile جدید و ست کردن PORT صحیح یا Port field در Container Settings، Restart مکرر حل شد

**۷. خطای مجدد بعد از Build و Push جدید**

* **دلیل:** ممکن بود Container Settings یا PORT تنظیم نشده باشد
* **راه‌حل:** بعد از هر Push بررسی کردیم Port و Identity، سپس App Service Restart شد

---

### 🔹 نکات کلیدی جمع‌بندی

1. Managed Identity + ACR → نقش **AcrPull** حتماً داده شود
2. Port داخلی کانتینر و ASP.NET Core هماهنگ باشد
3. App Service Plan حداقل **B1** برای کانتینر لازم است
4. Dockerfile باید پورت را صریحاً `EXPOSE` و `ASPNETCORE_URLS` ست کند
5. بعد از هر تغییر Container Settings یا Dockerfile، App Service را Restart کنید

---

💡 نتیجه نهایی: بعد از این تغییرات، کانتینر **Stable بالا می‌آید** و مسیر:

```
http://AzureDemoApiDocker-vahidfaraji.azurewebsites.net/api/hello
```

به درستی پاسخ `"Hello from AzureDemoApi!"` را می‌دهد ✅

