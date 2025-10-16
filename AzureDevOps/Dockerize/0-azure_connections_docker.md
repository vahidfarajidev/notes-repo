# Azure Service Connections

در Azure وقتی از اصطلاح **کانکشن** یا **Service Connection** استفاده می‌کنیم (مثلاً در **Azure DevOps**)، منظور راهی است که سیستم شما بتواند با منابع Azure یا سرویس‌های دیگر ارتباط برقرار کند. دو نوعی که رایج هستند:

---

## 1. Azure Resource Manager (ARM)

- **چیست؟**
  Azure Resource Manager یک لایه مدیریت برای منابع Azure است. وقتی کانکشن از نوع ARM می‌سازیم، به شما اجازه می‌دهد که منابع Azure مثل VM، Storage Account، App Service و غیره را مدیریت کنید.

- **کاربردها:**
  - ساخت و مدیریت Resource Group
  - استقرار (deploy) اپلیکیشن‌ها با **ARM Templates**
  - مدیریت دسترسی‌ها و سیاست‌ها

- **چطور کار می‌کند:**
  شما نیاز به یک **Service Principal** یا **Managed Identity** دارید تا DevOps یا ابزارهای دیگر بتوانند به Azure دسترسی امن پیدا کنند.

---

## 2. Docker Registry

- **چیست؟**
  Docker Registry یک سرویس برای ذخیره و مدیریت **Docker Images** است. Azure خودش سرویس مشابه دارد به اسم **Azure Container Registry (ACR)**.

- **کاربردها:**
  - Push و Pull کردن Docker Imageها
  - استفاده از Imageها در **AKS (Azure Kubernetes Service)** یا App Service
  - مدیریت نسخه‌های مختلف کانتینر

- **چطور کار می‌کند:**
  وقتی کانکشن به Docker Registry ایجاد می‌کنید، ابزارهای شما (مثلاً Azure DevOps Pipeline) می‌توانند مستقیم با Registry ارتباط برقرار کنند و Imageها را به آن بفرستند یا از آن دریافت کنند.

---

## Push کردن Docker Image به ACR

برای **push کردن Docker image به Azure Container Registry**، شما باید از **Docker Registry Connection** استفاده کنید، نه ARM Connection.

- **Docker Registry Connection**: برای احراز هویت و تعامل مستقیم با خود **ACR** جهت push/pull image لازم است.
- **ARM Connection**: فقط وقتی نیاز دارید منابع Azure را مدیریت کنید، مثل ایجاد یا تغییر یک ACR، به کار می‌رود.

پس اگر فقط می‌خواهید image را push کنید، کافی است یک **Docker Registry Connection** داشته باشید که به ACR شما متصل باشد.

