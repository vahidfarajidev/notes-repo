# راهنمای کامل دستورات .NET CLI برای پروژه‌های C#

این مقاله توضیح می‌دهد که دستورات `dotnet restore`, `dotnet build`, `dotnet test`, و `dotnet publish` چه کاری انجام می‌دهند و چطور می‌توان آنها را در محیط PowerShell بدون باز کردن Visual Studio استفاده کرد.

---

## 1. معرفی دستورات

### **dotnet restore**
- وظیفه: دانلود و آماده‌سازی تمام پکیج‌های NuGet مورد نیاز پروژه.
- معادل: **NuGet Package Manager → Restore** در Visual Studio.
- نکته: باید قبل از build اجرا شود تا پروژه بدون خطا ساخته شود.

### **dotnet build**
- وظیفه: کامپایل کردن کد پروژه.
- معادل: **Build Solution** در Visual Studio.
- نکته: اگر خطای کامپایل داشته باشد، build کامل نمی‌شود.

### **dotnet test**
- وظیفه: اجرای تست‌های تعریف‌شده در پروژه.
- معادل: **Test Explorer → Run All Tests** در Visual Studio.
- نکته: خطاهای تست روی فایل‌های ساخته شده تاثیر نمی‌گذارند ولی گزارش داده می‌شوند.

### **dotnet publish -c Release -o ./publish**
- وظیفه: آماده‌سازی پروژه برای انتشار در حالت Release و قرار دادن فایل‌ها در فولدر `publish`.
- معادل: **Right-click → Publish → Folder/Release** در Visual Studio.
- نکته: فقط وقتی `build` موفق باشد اجرا شود.

### توضیح کوتاه دستورات dotnet publish

- **-c** مخفف **Configuration** هست  
- مقدار Release یعنی پروژه رو در حالت Release بسازه نه Debug

- **-o** مخفف **Output** هست  
- ./publish یعنی یک پوشه به نام publish در مسیر جاری ساخته بشه و خروجی‌ها اونجا قرار بگیره

---

## 2. اجرا در محیط بدون Visual Studio

### **Developer PowerShell**
- این محیط همراه با Visual Studio نصب می‌شود و مسیرهای لازم برای .NET SDK و ابزارهای VS را بارگذاری می‌کند.
- بهترین و امن‌ترین گزینه برای اجرای دستورات بدون باز کردن Visual Studio است.

### **PowerShell یا Command Prompt معمولی**
- امکان اجرا وجود دارد، اما باید مطمئن شوید که مسیر دات‌نت SDK در PATH تنظیم شده باشد.
- بررسی نصب دات‌نت:
  ```powershell
  dotnet --version
  ```

### **مزایای اجرای خط فرمان**
- امکان اجرای خودکار مراحل پروژه بدون باز کردن VS.
- مناسب برای CI/CD pipelines مانند GitHub Actions و Azure DevOps.
- امکان اجرا با یک دستور واحد یا اسکریپت.

---

## 3. اجرای دستورات به ترتیب

### روش دونه‌دونه:
```powershell
dotnet restore
dotnet build
dotnet test
dotnet publish -c Release -o ./publish
```
- مزیت: مشاهده مرحله به مرحله خطاها و راحت‌تر رفع کردن مشکلات.

### روش پشت سر هم:
```powershell
dotnet restore; dotnet build; dotnet test; dotnet publish -c Release -o ./publish
```
- یا فقط اگر دستور قبلی موفق بود:
```powershell
dotnet restore && dotnet build && dotnet test && dotnet publish -c Release -o ./publish
```
- مزیت: اجرای خودکار مراحل بدون دخالت کاربر، با کنترل خطاها.

---

## 4. نتیجه‌گیری
این دستورات امکان اجرای تمامی عملیات **Build، Test و Publish** پروژه‌های C# را بدون نیاز به Visual Studio فراهم می‌کنند. با استفاده از PowerShell و اسکریپت‌ها می‌توان روند توسعه و انتشار را سریع، خودکار و قابل اتکا کرد.

---

