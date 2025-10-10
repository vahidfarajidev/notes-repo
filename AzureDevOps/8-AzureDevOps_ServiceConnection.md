# 🚀 مرحله ۴ — ایجاد Service Connection بین Azure DevOps و Azure

در این مرحله از فرایند **CI/CD**، هدف ما اینه که بین **Azure DevOps** و حساب **Azure** خودمون یک ارتباط امن برقرار کنیم تا بتونیم به صورت خودکار پروژه‌هامون رو روی Azure **Deploy** کنیم.

---

## 🎯 هدف این مرحله

به Azure DevOps اجازه می‌دیم که بدون نیاز به وارد کردن رمز یا تنظیمات دستی، بتونه:
- منابع Azure (مثل App Service، Storage، Function و ...) رو مدیریت کنه؛  
- و خروجی Pipeline رو مستقیماً روی Azure مستقر کنه.

این ارتباط امن رو **Service Connection** می‌نامیم.

---

## 🔹 گام ۴.۱ — ایجاد Service Connection

### ۱️⃣ ورود به تنظیمات پروژه
1. وارد **Azure DevOps** شو.  
2. پروژه‌ات رو باز کن (مثلاً: `AzureDemoProject`).  
3. از پایین سمت چپ صفحه، روی **⚙️ Project settings** کلیک کن.

---

### ۲️⃣ ساخت Service Connection جدید
1. در بخش **Pipelines**، گزینه‌ی **Service connections** رو انتخاب کن.  
2. روی دکمه‌ی **New service connection** کلیک کن.  
3. از لیست باز شده، گزینه‌ی **Azure Resource Manager** رو انتخاب کن.

---

### ۳️⃣ تنظیم نوع احراز هویت
1. در صفحه‌ی باز شده، نوع Authentication رو بذار روی:
   ```
   Service principal (automatic)
   ```
   🔒 این روش باعث می‌شه Azure DevOps خودش به‌صورت امن یک App Registration در Azure ایجاد کنه و نیازی به رمز یا تنظیم دستی نباشه.  
2. روی **Next** بزن.

---

### ۴️⃣ انتخاب Subscription و نام‌گذاری Connection
1. در مرحله‌ی بعد، اشتراک (Subscription) خودت رو از لیست انتخاب کن.  
2. در قسمت **Scope level** مقدار `Subscription` رو انتخاب کن (نه Resource Group).  
3. برای نام اتصال بنویس مثلاً:
   ```
   AzureDemo-Connection
   ```
4. روی **Verify and save** بزن ✅

---

## 🔒 نتیجه نهایی

الان Azure DevOps به اشتراک Azure تو متصل شده و می‌تونه از طریق همین Connection پروژه‌هات رو مستقر کنه.

از این به بعد، می‌تونیم در مراحل بعدی Pipeline با همین Connection پروژه‌مون رو Deploy کنیم (مثلاً روی Azure App Service).

---

## 🧠 نکته مهم

Service Connection در واقع **پل ارتباطی امن بین Azure DevOps و Azure** هست.  
به کمک این پل، هر Pipeline در DevOps می‌تونه بدون ورود اطلاعات محرمانه، پروژه‌ت رو در Azure مستقر کنه.

---

📍 بعد از ساخت Service Connection، آماده‌ایم برای **مرحله بعدی: ساخت App Service در Azure و تنظیم مرحله‌ی Deploy در Pipeline.**
