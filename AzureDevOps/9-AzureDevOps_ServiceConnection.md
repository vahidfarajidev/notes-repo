# ⚠️ رفع خطای "You don’t appear to have an active Azure subscription" در Azure DevOps

این خطا زمانی نمایش داده می‌شود که هنگام ایجاد **Service Connection** در Azure DevOps،  
سیستم نتواند هیچ **Azure Subscription (اشتراک فعال)** مرتبط با حساب کاربری تو پیدا کند.

---

## 🧩 معنی خطا

پیغام  
> **You don’t appear to have an active Azure subscription**  

یعنی حساب Microsoft / Azure که با آن وارد Azure DevOps شدی،  
به هیچ اشتراک فعالی در Azure متصل نیست یا دسترسی لازم به آن را ندارد.

بدون Subscription فعال، DevOps نمی‌تواند پروژه‌ها را روی Azure **Deploy** کند.

---

## ✅ راه‌حل گام‌به‌گام

### 🔹 گام ۱ — بررسی وجود Subscription در Azure Portal

1. وارد [Azure Portal](https://portal.azure.com) شو.  
2. در نوار جستجو بنویس **Subscriptions** و وارد بخش آن شو.  
3. در این صفحه باید حداقل یکی از موارد زیر را ببینی:
   - `Pay-As-You-Go`
   - `Azure for Students`
   - `Free Trial`
   - یا Subscription سازمانی شرکتت

📍 اگر هیچ آیتمی نمایش داده نمی‌شود، یعنی هنوز Subscription نداری.

---

### 🔹 گام ۲ — ساخت Subscription جدید (در صورت نداشتن اشتراک)

اگر حساب Azure نداری یا خالی است:

1. برو به [https://azure.microsoft.com/free](https://azure.microsoft.com/free)  
2. با همان حساب کاربری Microsoft که در DevOps استفاده می‌کنی وارد شو.  
3. یک اشتراک **Free Trial (۳۰ روزه)** بساز.  
   > شامل ۲۰۰ دلار اعتبار رایگان برای تست است.

پس از ایجاد، مجدداً به Azure DevOps برگرد و اتصال را دوباره امتحان کن.

---

### 🔹 گام ۳ — اگر Subscription داری ولی DevOps نمی‌بیند

در این حالت احتمالاً حساب DevOps به Subscription دسترسی ندارد.  
باید در Azure Portal برای حساب خودت نقش مناسب تنظیم کنی:

1. در Azure Portal برو به:  
   **Subscriptions → [نام اشتراک] → Access control (IAM)**
2. روی **Add role assignment** کلیک کن.
3. نقش زیر را بده:
   ```
   Role: Contributor
   Member: [حساب کاربری Microsoft که با آن در DevOps لاگین کردی]
   ```
4. روی **Save** بزن و چند دقیقه صبر کن.

بعد از این، DevOps باید بتواند Subscription را پیدا کند ✅

---

### 🔹 گام ۴ — Refresh در Azure DevOps

1. صفحه ساخت **Service Connection** را در DevOps ببند.  
2. دوباره باز کن و روی **Authorize / Sign in to Azure** کلیک کن.  
3. حالا Subscription جدید باید در لیست نمایش داده شود.

---

## 🧠 نکته پایانی

- هر Azure DevOps Project برای اتصال به Azure **حداقل به یک Subscription فعال نیاز دارد.**  
- اگر چند حساب Microsoft داری، حتماً با همان حسابی وارد شو که Subscription در آن ایجاد شده است.  
- پیشنهاد مایکروسافت: همیشه برای DevOps از حسابی با نقش **Contributor** یا **Owner** استفاده کن.

---

### ✅ نتیجه نهایی

با داشتن یک اشتراک فعال و مجوز مناسب،  
دکمه‌ی **“Verify and Save”** در ساخت Service Connection فعال می‌شود  
و DevOps می‌تواند به Azure متصل شود.

---

📘 **مستندات رسمی:**  
🔗 [Troubleshoot Azure Resource Manager service connections](https://learn.microsoft.com/en-us/azure/devops/pipelines/release/azure-rm-endpoint)
