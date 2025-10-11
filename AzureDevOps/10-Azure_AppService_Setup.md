# 🚀 مرحله ۵ — ساخت Azure App Service برای میزبانی Web API

## 🎯 هدف
ایجاد یک **App Service** در Azure برای اجرای خودکار برنامه‌ی Web API بعد از Build و Publish در Azure DevOps.

در واقع این مرحله دقیقاً مربوط به ساخت محیط هاستینگ (میزبانی) در Azure هست، یعنی جایی که بعد از بیلد و پابلیش، فایل‌های خروجی پروژه‌ت آپلود می‌شن و به صورت آنلاین در دسترس قرار می‌گیرن.

---

## 🧩 گام ۵.۱ — ایجاد App Service در Azure Portal

۱. وارد [Azure Portal](https://portal.azure.com) شو.  
۲. از نوار کناری، گزینه‌ی **App Services** را انتخاب کن.  
۳. روی **➕ Create** کلیک کن.  
۴. در بخش **Basics** تنظیمات زیر را انجام بده:

| فیلد | مقدار پیشنهادی | توضیح |
|------|----------------|-------|
| **Subscription** | همان اشتراک Azure که در Service Connection انتخاب کردی | برای اتصال DevOps به Azure |
| **Resource Group** | `AzureDemo-RG` | یا ساخت Resource Group جدید |
| **Name** | `AzureDemo` | نام اپلیکیشن (در URL هم استفاده می‌شود) |
| **Publish** | `Code` | چون برنامه‌ی ما با کد دات‌نت پابلیش می‌شود |
| **Runtime stack** | `.NET 9 (یا 8 LTS)` | نسخه‌ی دات‌نت پروژه‌ات |
| **Operating System** | `Windows` | می‌توانی Linux هم انتخاب کنی |
| **Region** | نزدیک‌ترین منطقه (مثلاً `West Europe`) | برای کارایی بهتر |

۵. روی **Review + Create** بزن.  
۶. بعد از تأیید، روی **Create** کلیک کن.  

⏳ چند دقیقه صبر کن تا Deployment کامل شود.  

---

## ✅ نتیجه

الان یک App Service داری با آدرسی شبیه زیر:

```
https://<AppServiceName>.azurewebsites.net
```

فعلاً فقط صفحه‌ی پیش‌فرض Azure نمایش داده می‌شود،  
اما در **مرحله‌ی بعدی (Deployment Pipeline)** فایل‌های پابلیش‌شده از Azure DevOps به‌صورت خودکار در این App Service مستقر خواهند شد.

---

## 💡 نکته
App Service در واقع مثل یک “سرور وب ابری” عمل می‌کند که فایل‌های Build شده‌ی پروژه‌ات را اجرا می‌کند — بدون نیاز به نصب IIS یا پیکربندی دستی سرور.

---

## 🔜 مرحله بعد
تنظیم **Continuous Deployment (CD)** برای ارسال اتوماتیک Buildها از Azure DevOps به App Service.
