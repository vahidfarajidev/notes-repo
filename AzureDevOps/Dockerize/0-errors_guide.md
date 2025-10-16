# راهنمای خطاهای Pipeline Docker/ACR و راه‌حل‌های نهایی

این مستند شامل خطاهای رایج هنگام اجرای Pipeline برای Build و Push Docker Image به Azure Container Registry (ACR) و راه‌حل‌های نهایی آن‌ها است.

---

## خطا ۱ — Service Connection پیدا نشد
```
The pipeline is not valid. Job Job: Step Docker2 input containerRegistry references service connection ... which could not be found.
```
**علت:**
- در YAML قبلی، `dockerRegistryServiceConnection` اشتباه بود یا Service Connection درست ایجاد نشده بود. در واقع Pipeline به یک Service Connection اشاره می‌کرد که وجود نداشت یا Authorization نداشت.

**راه حل نهایی:**
- ساخت یک Service Connection از نوع Docker Registry با نام AzureDemo-ACR.
- انتخاب درست **Login Server** از ACR یعنی Login Server: azuredemoacr2873.azurecr.io
- Username و Password: از ACR گرفته شد.
- فایل YAML به‌روزرسانی شد تا از این Service Connection استفاده کند:
  
```yaml
dockerRegistryServiceConnection: 'AzureDemo-ACR'
containerRegistry: azuredemoacr2873.azurecr.io
```
---

## خطا ۲ — انتخاب Agent اشتباه (Windows) برای .NET 9 Docker Images
```
no matching manifest for windows/amd64 ... mcr.microsoft.com/dotnet/sdk:9.0
```
**علت:**
- تصاویر .NET 9 برای Linux ساخته شده‌اند و روی Windows agent در Azure DevOps قابل Pull نیستند.

**راه حل نهایی:**
- تغییر **pool** به Linux agent:
```yaml
vmImage: 'ubuntu-latest'
```
- اجرای Build و Push روی Agent لینوکسی

---

## خطا ۳ — dotnet restore پروژه پیدا نشد
```
MSBUILD : error MSB1003: Specify a project or solution file.
```
**علت:**
- Dockerfile مسیر پروژه (`.csproj`) را درست مشخص نکرده بود و دستور `dotnet restore` در دایرکتوری اشتباه اجرا می‌شد.

**راه حل نهایی:**
- اصلاح Dockerfile:
```dockerfile
COPY AzureDemoApi/AzureDemoApi.csproj ./AzureDemoApi/
RUN dotnet restore ./AzureDemoApi/AzureDemoApi.csproj
```
- مسیر دقیق پروژه برای restore مشخص شد و بعد کل سورس Copy شد

---

## خطا ۴ — Push به ACR انجام نشد
```
An image does not exist locally with the tag: azuredemoacr2873.azurecr.io/azuredemoapi
```
**علت:**
- tag یا repository اشتباه تعریف شده بود
- Build و Push از tag مشابه نبود یا مسیر کامل Login Server استفاده نشده بود

**راه حل نهایی:**
- استفاده از **tag کامل** در Build و Push:
```yaml
tags: $(containerRegistry)/$(imageRepository):$(tag)
```
- `dockerRegistryServiceConnection` درست و معتبر باشد
- اطمینان از اینکه Build Docker Image قبل از Push موفق اجرا شده است

---

## خطا ۵ — Overwrite Image در ACR
```
هر بار Pipeline اجرا می‌شود، Image با تگ ثابت overwrite می‌شود
```
**راه حل نهایی:**
- استفاده از **tag پویا** بر اساس Build ID:
```yaml
tag: '1.0.$(Build.BuildId)'
```
- این کار باعث می‌شود هر Build یک Image جدید داشته باشد و نسخه قبلی پاک نشود

---

## ✅ جمع‌بندی
- Dockerfile مسیر پروژه را درست مشخص کرده
- Agent لینوکسی برای سازگاری با .NET 9 استفاده شده
- Service Connection به شکل صحیح ساخته شده
- Tag پویا برای Versioning و Rollback ایجاد شده

با این اصلاحات، Pipeline **Build و Push Docker Image به ACR** بدون مشکل اجرا می‌شود.

