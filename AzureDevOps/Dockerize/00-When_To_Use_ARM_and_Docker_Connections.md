# Azure DevOps Pipeline: ARM Connection و Docker Registry Connection

## ۱️⃣ ARM Connection چیست و چه زمانی لازم است؟
- **Azure Resource Manager (ARM) Connection** در Azure DevOps برای مدیریت منابع Azure استفاده می‌شود.
- **زمان نیاز:** وقتی pipeline شما قرار است **منابع Azure بسازد یا مدیریت کند** (مثل Resource Group، ACR، VM، Storage).  
- **چرایی نیاز:** وقتی pipeline یک **Resource Group یا ACR جدید می‌سازد**، با APIهای مدیریتی Azure تماس می‌گیرد و نیاز به **دسترسی مدیریتی** دارد. Docker Registry Connection تنها اجازه push/pull image می‌دهد و نمی‌تواند منابع جدید بسازد.  
- **زمان لازم نیست:** وقتی فقط می‌خواهید Docker image را push کنید به یک **ACR موجود** (چه ساخته شده داخل pipeline باشد چه قبلاً روی Developer Powershell ساخته شده باشد). در این حالت فقط **Docker Registry Connection** کافی است.

## ۲️⃣ سطح‌های دسترسی ARM Connection
| سطح دسترسی | چه چیزی را مدیریت می‌کند | چه زمانی استفاده می‌شود |
|------------|------------------------|-----------------------|
| **Subscription-level** | همه Resource Groupها و منابع داخل Subscription | وقتی pipeline می‌خواهد RG یا منابع جدید بسازد یا منابع مختلف Subscription را مدیریت کند |
| **Resource Group-level** | فقط منابع داخل یک RG مشخص | وقتی pipeline فقط منابع داخل یک RG را مدیریت می‌کند، امنیت بالاتر و دسترسی محدود |

**نکته:** اگر ARM Connection شما Subscription-level باشد، **نیازی به انتخاب RG جداگانه نیست**. فقط وقتی محدود به RG است، باید RG مشخص شود.

## ۳️⃣ Docker Registry Connection
- برای push/pull Docker image به **Azure Container Registry (ACR)**.
- **ARM Connection لازم نیست** اگر فقط این کار را انجام می‌دهید.
- Pipeline شما فقط به این Service Connection نیاز دارد تا بتواند imageها را build و push کند.

## ۴️⃣ سناریوهای ترکیبی با pipeline
| سناریو | ARM Connection لازم؟ | توضیح |
|--------|--------------------|-------|
| Push Docker Image به **ACR موجود** که **روی Developer Powershell ساخته شده** | ❌ نه | فقط Docker Registry Connection کافی است، pipeline هیچ منبع جدیدی نمی‌سازد |
| Push Docker Image به **ACR موجود** داخل pipeline | ❌ نه | فقط Docker Registry Connection کافی است |
| ایجاد Resource Group و ACR **داخل pipeline** | ✅ بله | ARM Connection با دسترسی Subscription-level یا RG-level لازم است، چون pipeline دارد با APIهای مدیریتی Azure منابع جدید می‌سازد |
| مدیریت منابع داخل یک RG موجود | ✅ فقط اگر ARM Connection محدود به RG باشد | اگر Subscription-level است، دیگر نیاز نیست |

## ۵️⃣ نکات عملی و مهم
1. اگر RG و ACR را قبلاً **روی Developer Powershell ساخته‌اید**، pipeline فقط نیاز دارد **Docker Registry Connection** برای push.
2. اگر می‌خواهید pipeline خودش RG یا ACR بسازد، باید **ARM Connection با دسترسی مناسب** داشته باشید.
3. سطح دسترسی ARM Connection تعیین می‌کند که آیا باید RG را در connection مشخص کنید یا نه.
4. Docker Registry Connection هیچ دسترسی مدیریتی به Azure ندارد و فقط برای push/pull image استفاده می‌شود.

---

💡 **جمع‌بندی تک خطی نهایی:**  
- **Push image به ACR موجود (چه ساخته شده توسط pipeline باشد، چه روی Developer Powershell) → فقط Docker Registry Connection لازم است.**  
- **ساخت یا مدیریت منابع Azure داخل pipeline → ARM Connection لازم است. سطح دسترسی مشخص می‌کند که RG باید تعیین شود یا نه.**

