# راهنمای کامل: از نوشتن پروژه Dockerized C# تا استقرار در Azure با CI/CD

**خلاصه:** این مقاله مسیر End‑to-End را قدم‌به‌قدم توضیح می‌دهد: از نوشتن یک برنامه‌ی Dockerized C# شروع می‌کنیم، آن را در Git قرار می‌دهیم، با استفاده از **Azure DevOps** یک Pipeline برای Build و Push Docker Image به ACR می‌سازیم، سپس Image را به App Service مستقر می‌کنیم و در نهایت مانیتورینگ و Alerts فعال می‌کنیم. مراحل توسعه و گسترش آینده هم به صورت نام برده شده است.

---

## ۱. مقدمه — چرا این مسیر مهم است؟

* DevOps و CI/CD باعث می‌شوند نرم‌افزار سریع‌تر، امن‌تر و قابل‌اطمینان‌تر منتشر شود.
* استفاده از Docker با .NET 9 و Azure باعث **تکرارپذیری محیط** و **قابلیت جابجایی بین محیط‌ها** می‌شود.

---

## ۲. مفاهیم پایه‌ای: DevOps، Azure DevOps و Docker

* **DevOps:** ترکیب توسعه نرم‌افزار و عملیات زیرساخت برای تحویل سریع.
* **Azure DevOps:** ابزار CI/CD شامل Repos, Pipelines, Artifacts, Boards.
* **Dockerization:** تبدیل پروژه به Docker Image برای Build و Deploy یکسان در همه محیط‌ها.
* **ACR (Azure Container Registry):** مخزن امن برای ذخیره Docker Imageها.

---

## ۳. مسیر یادگیری و کار عملی — گام‌به‌گام

### مرحله ۱ — نوشتن پروژه C# Dockerized

* ابزار: **Visual Studio** یا **VS Code**
* نوع پروژه: **ASP.NET Core Web API**
* کنترلر ساده: `HelloController` با خروجی `"Hello from AzureDemoApi!"`
* HTTPS فعال، Docker Support فعال
* پروژه روی **.NET 9**

### مرحله ۲ — آماده‌سازی پروژه برای Docker و Git

* پروژه زیر کنترل نسخه Git
* شاخه اصلی: `main`
* فایل **Dockerfile** ساخته شده و Build لوکال تست شده
* Docker Image لوکال تست شده با `docker run` ✅

### مرحله ۳ — ساخت Azure DevOps Account و Project

* ایجاد Organization و Project در Azure DevOps
* Repository ایجاد و Push کد به Azure DevOps (`AzureDemoApi`)

### مرحله ۴ — ایجاد Service Connectionها

* **Azure Resource Manager Service Connection** → اتصال به Subscription Azure
* **Docker / ACR Service Connection** → دسترسی به Azure Container Registry

### مرحله ۵ — ساخت Azure Container Registry (ACR)

* ACR ایجاد شد (`azuredemoacr2873`)
* Docker Image لوکال Push شد به ACR

### مرحله ۶ — ایجاد Pipeline برای Build و Push Docker Image

* YAML Pipeline ساخته شد (`azure-pipelines-docker.yml`)
* مراحل:
  1. Build Docker Image
  2. Push Docker Image به ACR
* تگ فعلاً ثابت روی `1.0.0`

### مرحله ۷ — Deploy به Azure App Service

* App Service با نام `AzureDemoApiDocker-vahidfaraji` ساخته شد
* Pipeline Task `AzureWebAppContainer@1` برای Deploy اضافه شد
* Docker Image مستقر و تست نهایی انجام شد (`/api/hello`)

---

## ۴. مرحله‌های پیشنهادی و گسترش حرفه‌ای (نام برده)

* **Tag پویا و Versioning:** استفاده از `$(Build.BuildId)` یا تاریخ برای جلوگیری از overwrite
* **Environment Variables و Secrets:** استفاده از Azure Key Vault برای مدیریت امن مقادیر محرمانه
* **Multi-Stage Pipeline:** Build → Test → Push → Deploy Dev → Deploy Staging → Deploy Prod
* **Monitoring پیشرفته:** Application Insights، Log Analytics، Alerts و Notifications
* **Docker Compose / Multi-container support:** در صورت اضافه شدن سرویس‌های دیگر
* **Kubernetes / AKS Deployment:** برای مقیاس‌پذیری و مدیریت چند سرویس
* **Infrastructure as Code (IaC):** ARM/Bicep/Terraform برای ساخت خودکار منابع
* **Security و Compliance:** مدیریت Role، Secrets، Vulnerability Scanning
* **Automated Tests / Integration Tests:** قبل و بعد از Deploy برای اطمینان از کیفیت

---

## ۵. چک‌لیست نهایی End-to-End

1. [ ] پروژه C# Dockerized با HTTPS آماده و تست شده (`HelloController`)
2. [ ] پروژه زیر کنترل Git قرار دارد، شاخه `main` آماده است
3. [ ] Dockerfile ساخته و لوکال تست شده
4. [ ] Repository در Azure DevOps ایجاد و Push شده
5. [ ] Service Connectionها برای Azure و Docker / ACR ایجاد شده
6. [ ] Azure Container Registry ساخته شده و Image Push شده
7. [ ] Pipeline YAML شامل Build, Push Docker Image و Deploy به App Service است
8. [ ] Deploy اتوماتیک Pipeline موفق بوده و API آنلاین اجرا می‌شود
9. [ ] Application Insights فعال و Telemetry جمع‌آوری می‌شود
10. [ ] Alerts و Notifications برای خطاها و Performance تنظیم شده‌اند
11. [ ] Optional: مراحل گسترش حرفه‌ای مثل Multi-Stage Pipeline، IaC، AKS و Security آماده‌اند

---

## ۶. متن پیشنهادی برای پست LinkedIn

> امروز مسیر کامل از **نوشتن یک پروژه Dockerized C#** تا **استقرار آن در Azure با CI/CD** را پیاده‌سازی کردم.
> از Azure DevOps برای Build و Push Docker Image و Deploy اتوماتیک به App Service استفاده کردم.
> این مسیر، تجربه عملی برای Build، Versioning، Deployment و Monitoring خودکار در Azure بود.
> #DotNet #Azure #DevOps #Docker #CSharp #CI_CD

