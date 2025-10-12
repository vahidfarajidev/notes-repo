# راهنمای کامل: از نوشتن برنامه C# تا استقرار در Azure با CI/CD

**خلاصه:** این مقاله مسیر End‑to-End را قدم‌به‌قدم توضیح می‌دهد: از نوشتن یک برنامه‌ی C# شروع می‌کنیم، آن را در Git قرار می‌دهیم، با استفاده از **Azure DevOps** یک Pipeline برای CI می‌سازیم، سپس با CD آن را در Azure مستقر و عملیاتی می‌کنیم و در نهایت مانیتورینگ و Alerts فعال می‌کنیم. در انتها چک‌لیست نهایی برای مرور سریع ارائه شده است.

---

## ۱. مقدمه — چرا این مسیر مهم است؟
تحویل مداوم (Continuous Delivery) و DevOps به ما کمک می‌کنند نرم‌افزار را سریع‌تر، قابل‌اطمینان‌تر و با بازخورد سریع‌تر منتشر کنیم. برای یک توسعه‌دهنده‌ی C#، استفاده از **Azure** و **Azure DevOps** منفعت‌های زیادی دارد: یکپارچگی خوب با .NET و Visual Studio، ابزارهای CI/CD آماده، و سرویس‌های مدیریت‌شده برای میزبانی، مانیتورینگ و امنیت.

---

## ۲. مفاهیم پایه‌ای: DevOps و Azure DevOps
**DevOps** = ترکیب توسعهٔ نرم‌افزار + عملیات زیرساخت برای تحویل سریع و با کیفیت.

**Azure DevOps** مجموعه‌ای از سرویس‌ها برای پیاده‌سازی DevOps است:
- **Repos** — مدیریت سورس کد (Git)؛
- **Pipelines** — اتوماسیون Build و Release (CI/CD)؛
- **Artifacts** — مدیریت پکیج‌ها و وابستگی‌ها؛
- **Boards** — پیگیری تسک‌ها و مدیریت فرایند توسعه.

> در عمل «Dev.Azure» معمولاً به همان Azure DevOps اشاره دارد — ابزار اصلی مایکروسافت برای پیاده‌سازی CI/CD در اکوسیستم Azure.

---

## ۳. مسیر یادگیری و کار عملی — گام‌به‌گام
### مرحله ۱ — نوشتن برنامهٔ C#
- ابزار: **Visual Studio** یا **VS Code**؛
- نوع پروژه پیشنهادی: **ASP.NET Core Web API** یا **Console App**؛
- کنترلر ساده: `HelloController` با خروجی `"Hello from AzureDemoApi!"`؛
- HTTPS فعال و Docker Support در این مرحله لازم نیست.

### مرحله ۲ — آماده‌سازی برای CI/CD
- پروژه زیر کنترل نسخه Git قرار گیرد و `.gitignore` مناسب داشته باشد؛
- شاخه اصلی: `main` و در صورت نیاز شاخه‌های `develop` یا feature branches برای توسعه؛
- پروژه با CLI قابل Build و Publish باشد (`dotnet restore/build/test/publish`)؛
- Optional: Dockerization بعداً در صورت نیاز.

### مرحله ۳ — ساخت Azure DevOps Account و Project
- ساخت Organization و Project در [Azure DevOps](https://dev.azure.com/)؛
- Repository ایجاد و Push کد به Azure DevOps؛
- توصیه: نام Repository `AzureDemoApi` و Project روی Azure `AzureDemo`.

### مرحله ۴ — ایجاد Service Connection
- ایجاد **Service Connection** از نوع Azure Resource Manager برای اتصال Azure DevOps به Subscription Azure؛
- نام Connection پیشنهادی: `AzureDemo-Connection`.

### مرحله ۵ — ساخت App Service در Azure
- ایجاد App Service با نام `AzureDemoApi-vahidfaraji`؛
- Runtime stack: `.NET 8` یا نسخه پروژه؛
- OS: Windows و Region نزدیک.

### مرحله ۶ — اضافه کردن مرحله‌ی Deploy به Pipeline
- ویرایش فایل `azure-pipelines.yml` و اضافه کردن Task `AzureWebApp@1` با Service Connection و App Service موردنظر؛
- Commit و Push برای اجرای Pipeline و Deploy خودکار.

### مرحله ۷ — تست نهایی Web API
- URL App Service را پیدا کرده و endpoint `/api/hello` را بررسی کن؛
- باید خروجی `"Hello from AzureDemoApi!"` را مشاهده کنی.

### مرحله ۸ — فعال‌سازی Application Insights
- ایجاد Application Insights و اتصال آن به App Service؛
- بررسی Live Metrics و Telemetry.

### مرحله ۹ — تنظیم Alerts و Notifications
- تعریف Alert Rule در Application Insights برای Response Time، Failed Requests و Exceptions؛
- اتصال Action Group برای دریافت Notification (Email/Teams).

### مرحله ۱۰ — بهینه‌سازی CI/CD و Deployment
- Branching Strategy مناسب (main, develop, feature branches)؛
- Multi-Stage Pipeline: Dev → Staging → Prod با approvals؛
- Secrets در Key Vault و دسترسی امن در Pipeline؛
- Infrastructure as Code با ARM/Bicep/Terraform؛
- Automated Tests قبل و بعد از Deploy؛
- Artifact Versioning و امکان Rollback؛
- Optional: Dockerization برای مقیاس‌پذیری آینده.

---

## ۴. چک‌لیست نهایی End-to-End
1. [ ] پروژه C# با HTTPS آماده و تست شده است (`HelloController`)  
2. [ ] پروژه زیر کنترل Git قرار دارد، شاخه `main` آماده است  
3. [ ] پروژه بدون خطا با CLI Build و Publish می‌شود  
4. [ ] Repository در Azure DevOps ایجاد و Push شده (`AzureDemoApi`)  
5. [ ] Service Connection به Azure ایجاد شده (`AzureDemo-Connection`)  
6. [ ] App Service در Azure ساخته شده (`AzureDemoApi-vahidfaraji`)  
7. [ ] Pipeline YAML شامل مراحل Build, Test, Publish و Deploy به App Service است  
8. [ ] Deploy اتوماتیک Pipeline موفق بوده و API آنلاین اجرا می‌شود  
9. [ ] Application Insights فعال شده و Telemetry جمع‌آوری می‌شود  
10. [ ] Alerts و Notifications برای خطاها و Performance تنظیم شده‌اند  
11. [ ] Best Practices برای Branching, Environment Stages, Secrets Management, IaC و Automated Tests رعایت شده است  
12. [ ] Optional: Dockerization آماده برای آینده و مقیاس‌پذیری

---

## ۵. متن پیشنهادی برای پست LinkedIn
> امروز مسیر کامل از **نوشتن یک برنامه C#** تا **استقرار آن در Azure با CI/CD** را یاد گرفتم. از Visual Studio و Azure DevOps برای پیاده‌سازی CI/CD استفاده کنید: build، تست خودکار، و deployment به App Service — همه به‌صورت خودکار. اگر می‌خواهید، نمونه‌ی YAML pipeline و یک راهنمای گام‌به‌گام آماده کردم. #DotNet #Azure #DevOps #CSharp

