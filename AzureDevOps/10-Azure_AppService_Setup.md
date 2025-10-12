# 🚀 مرحله ۵ — ساخت Azure App Service برای میزبانی Web API

## 🎯 هدف
ایجاد یک **App Service** در Azure برای اجرای خودکار برنامه‌ی Web API بعد از Build و Publish در Azure DevOps.

در واقع این مرحله دقیقاً مربوط به ساخت محیط هاستینگ (میزبانی) در Azure هست؛ یعنی جایی که بعد از بیلد و پابلیش، فایل‌های خروجی پروژه‌ات آپلود می‌شن و به صورت آنلاین در دسترس قرار می‌گیرن.

---

## 🧩 گام ۵.۱ — ایجاد App Service در Azure Portal

۱. وارد [Azure Portal](https://portal.azure.com) شو.  
۲. از نوار کناری، گزینه‌ی **App Services** را انتخاب کن.  
۳. روی **➕ Create** کلیک کن.

در این مرحله چند نوع App Service برای ساخت بهت نمایش داده می‌شود:

- **Web App** → برای برنامه‌های وب و API (انتخاب این گزینه ✅)
- **Static Web App** → برای سایت‌های استاتیک (HTML/CSS/JS)
- **Web App + Database** → برای اپ‌هایی که همراه با دیتابیس ساخته می‌شن
- **WordPress on App Service** → مخصوص نصب وردپرس روی Azure

✅ ما باید گزینه‌ی **Web App** را انتخاب کنیم.

---

### 🧱 تنظیمات بخش **Basics**

در بخش **Basics** تنظیمات زیر را انجام بده:

| فیلد | مقدار پیشنهادی | توضیح |
|------|----------------|-------|
| **Subscription** | همان اشتراک Azure که در Service Connection انتخاب کردی | برای اتصال DevOps به Azure |
| **Resource Group** | `AzureDemo-RG` | یا ساخت Resource Group جدید |
| **Name** | `AzureDemo` *(در صورت تکراری بودن، نام یکتا مثل `AzureDemoApiVahid` انتخاب کن)* | نام اپلیکیشن (در URL هم استفاده می‌شود) |
| **Publish** | `Code` | چون برنامه‌ی ما با کد دات‌نت پابلیش می‌شود |
| **Runtime stack** | `.NET 9` یا `.NET 8 (LTS)` | نسخه‌ی دات‌نت پروژه‌ات |
| **Operating System** | `Windows` | در صورت نیاز می‌توانی `Linux` هم انتخاب کنی |
| **Region** | نزدیک‌ترین منطقه (مثلاً `West Europe` یا `Germany West Central`) | برای کارایی بهتر و تاخیر کمتر |

---

### 🧾 سایر تنظیمات

اگر خواستی می‌تونی تنظیمات بخش‌های دیگر مثل **Monitoring** یا **Deployment** را فعلاً در حالت پیش‌فرض رها کنی — بعداً از طریق Azure DevOps به صورت خودکار تنظیم خواهند شد.

---

### 🚀 ساخت نهایی

۱. روی **Review + Create** بزن.  
۲. بعد از بررسی، روی **Create** کلیک کن.

⏳ چند دقیقه صبر کن تا Deployment کامل شود.

---

## ✅ نتیجه

اگر همه چیز درست پیش رفته باشد، پیامی مانند زیر نمایش داده می‌شود:

> **“Your deployment is complete”**

و دکمه‌ای به نام **Go to Resource** در پایین صفحه ظاهر می‌شود.

روی **Go to Resource** کلیک کن تا وارد صفحه‌ی App Service شوی. در این بخش می‌تونی اطلاعات زیر را ببینی:

| بخش | توضیح |
|------|-------|
| **Overview** | آدرس اینترنتی برنامه (URL) مثل `https://azuredemoapi.azurewebsites.net` |
| **Essentials** | مشخصات پلن، Resource Group، Region و اشتراک |
| **Deployment Center** | محل اتصال Azure DevOps برای استقرار خودکار |
| **Configuration** | تنظیمات محیطی و App Settings |
| **Log Stream** | مشاهده زنده‌ی لاگ‌های اجرای برنامه |
| **Browse** | باز کردن اپ در مرورگر (فعلاً صفحه پیش‌فرض Azure نمایش داده می‌شود) |

---

## 💡 نکته مهم

در واقع **App Service مثل یک سرور وب ابری** عمل می‌کند که فایل‌های Build شده‌ی پروژه‌ات را اجرا می‌کند — بدون نیاز به نصب IIS یا پیکربندی دستی سرور.

الان App Service آماده‌ست ✅  
فقط باید Pipeline (از Azure DevOps) رو بهش متصل کنیم تا هر بار که Build موفق انجام میشه، خروجی به‌صورت خودکار روی همین App Service Deploy بشه.

---

## 🔜 مرحله بعد

در مرحله‌ی بعدی، می‌ریم سراغ **تنظیم Continuous Deployment (CD)**  
تا Buildهای تولید شده در Azure DevOps به صورت خودکار به این App Service ارسال و مستقر شوند. 🚀
