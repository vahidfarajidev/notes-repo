# 🚀 تنظیم Alerts و Notification در Azure

حالا بریم فقط **یک قدم بعدی** — یعنی **تنظیم Alerts و Notification برای خطاها و مشکلات احتمالی** تا وقتی اتفاق غیرمنتظره‌ای افتاد سریع مطلع بشی.

---

## 🔹 مرحله ۹ — تنظیم Alerts در Azure

### 🎯 هدف این قدم

اگر API خطا داد، Response Time بالا رفت یا Availability کاهش پیدا کرد، یک Alert به تو ارسال شود تا سریع واکنش نشان دهی.

---

### گام ۹.۱ — ایجاد Alert Rule در Azure

1. وارد **Azure Portal** شو
2. به **Application Insights → AzureDemoApi-Insights** برو
3. از منوی سمت چپ روی **Alerts → New alert rule** کلیک کن
4. Resource: همون Application Insights را انتخاب کن
5. Condition: نوع Alert را انتخاب کن، مثل:

   * **Server response time** (Performance)
   * **Request failed** (Exceptions)
6. Action Group: اگر قبلاً نداری، **Create action group** → Email یا Teams notification اضافه کن
7. Alert rule name: مثلاً `HighResponseTimeAlert`
8. Review + Create ✅

---

### گام ۹.۲ — بررسی عملکرد Alerts

* وقتی یک خطا یا شرایط تعریف‌شده رخ دهد، تو یا تیم‌ت از طریق Email یا Teams مطلع خواهید شد
* می‌توانی Thresholds را بعداً تنظیم کنی تا فقط در شرایط بحرانی Notification ارسال شود

---

✅ الان Alerts و Notifications فعال شدند و می‌توانی API را با خیال راحت مانیتور کنی.

---

✋ تا همین‌جا توقف می‌کنیم.  
وقتی گفتی **«برو قدم بعدی»**، می‌ریم سراغ **بهینه‌سازی و نکات حرفه‌ای برای CI/CD و Azure Deployment**.

