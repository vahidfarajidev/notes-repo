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
- تصاویر .NET 9 برای Linux ساخته شده‌اند و روی Windows agent در Azure DevOps قابل Pull نیستند یا Docker Agent روی Windows بود ولی base image .NET 9 فقط روی Linux موجود بود (mcr.microsoft.com/dotnet/aspnet:9.0).

**راه حل نهایی:**
- تغییر **pool** به Linux agent:
```yaml
pool:
vmImage: 'ubuntu-latest'
```
- اجرای Build و Push روی Agent لینوکسی یا این باعث شد Docker بتواند image لینوکسی را Pull و Build کند.

---

## خطا ۳ — dotnet restore پروژه پیدا نشد
```
MSBUILD : error MSB1003: Specify a project or solution file. The current working directory does not contain a project or solution file.
```
**علت:**
- در واقع Dockerfile مسیر پروژه (`.csproj`) را درست مشخص نکرده بود و دستور `dotnet restore` در دایرکتوری اشتباه اجرا می‌شد. یا مسیر کاری در Dockerfile اشتباه بود یا .csproj در مسیر Build context نبود.

**راه حل نهایی:**
- اصلاح Dockerfile تا مسیر پروژه و csproj درست تنظیم شود و COPY فایل‌ها به مسیر مناسب انجام شود.
```dockerfile
COPY AzureDemoApi/AzureDemoApi.csproj ./AzureDemoApi/
RUN dotnet restore ./AzureDemoApi/AzureDemoApi.csproj
```
- مسیر دقیق پروژه برای restore مشخص شد و بعد کل سورس Copy شد

---

## خطا ۴ — Push Image به ACR انجام نشد
```
An image does not exist locally with the tag: azuredemoacr2873.azurecr.io/azuredemoapi
```
**علت:**
- tag یا repository اشتباه تعریف شده بود یا Docker Build موفق شده بود ولی Tag دادن به image یا نام repository اشتباه بود.
- Build و Push از tag مشابه نبود یا مسیر کامل Login Server استفاده نشده بود

**راه حل نهایی:**
- استفاده از **tag کامل** در Build و Push:
```yaml
tags: $(containerRegistry)/$(imageRepository):$(tag)
```
- `dockerRegistryServiceConnection` درست و معتبر باشد
- اطمینان از اینکه Build Docker Image قبل از Push موفق اجرا شده است

---

## ✅ جمع‌بندی
- Dockerfile مسیر پروژه را درست مشخص کرده
- Agent لینوکسی برای سازگاری با .NET 9 استفاده شده
- Service Connection به شکل صحیح ساخته شده
با این اصلاحات، Pipeline **Build و Push Docker Image به ACR** بدون مشکل اجرا می‌شود.

صفحه Troubleshoot pipeline runs خطاها در سایت مایکروسافت:
https://learn.microsoft.com/en-us/azure/devops/pipelines/troubleshooting/troubleshooting?view=azure-devops


