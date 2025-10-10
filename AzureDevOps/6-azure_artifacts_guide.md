# فهم Artifact در Azure DevOps

در دنیای **CI/CD**، مفهوم **Artifact** نقش کلیدی دارد. این مقاله به زبان ساده توضیح می‌دهد Artifact چیست، چرا مهم است و چطور در Azure DevOps استفاده می‌شود.

---

## 🧱 1. Artifact چیست؟

در واقع Artifact همان **خروجی نهایی مرحله‌ی Build** است که برای مراحل بعدی (مثل Deploy) قابل استفاده است.  
به طور مثال در پروژه‌های .NET خروجی‌ها می‌توانند شامل موارد زیر باشند:

- فایل‌های DLL و EXE
- فایل‌های publish-ready برای deployment
- فایل‌های پیکربندی (appsettings.json و غیره)
- هر فایل دیگری که برای اجرای اپلیکیشن لازم است

> به بیان ساده: Artifact همان "بسته‌ی آماده برای ارسال" است.

---
## 🗂 مشاهده و دانلود Artifact

بعد از اجرای Pipeline:
1. وارد **Azure DevOps → Pipelines → Runs** شوید.  
2. روی اجرای مورد نظر کلیک کنید.  
3. در بخش **Summary**، لینک **1 published** مربوط به Artifact را مشاهده می‌کنید.  
4. با کلیک روی این لینک می‌توانید:
   - محتویات Artifact را ببینید.
   - یا فایل ZIP آن را دانلود کنید.
---
  

## 🧩 2. مسیر Artifact Staging Directory

در Azure DevOps، pipeline برای ذخیره‌سازی موقت خروجی build از یک مسیر پیش‌فرض استفاده می‌کند:

```
$(Build.ArtifactStagingDirectory)
```

این مسیر یه پوشه موقته روی ماشین build agent (مثلاً Ubuntu VM) که Azure DevOps برایت می‌سازه. Pipeline توی اون پوشه فایل‌های خروجی build رو جمع می‌کنه.

در YAML ما این خط باعثش می‌شه:

```yaml
- script: dotnet publish --configuration Release --no-build -o $(Build.ArtifactStagingDirectory)
  displayName: 'dotnet publish'
```

این دستور پروژه را **publish** می‌کند و خروجی نهایی را در پوشه staging قرار می‌دهد.

---

## 📦 3. انتشار Artifact

پس از build و publish، فایل‌ها باید برای مراحل بعدی نگهداری شوند. در Azure DevOps این کار با **PublishBuildArtifacts** انجام می‌شود. در واقع فقط build کردن کافی نیست — چون Azure DevOps بعد از اتمام job اون ماشین رو پاک می‌کنه! برای اینکه فایل‌های ساخته‌شده رو نگه‌داریم (مثلاً برای Deploy یا Download)، باید بگیم:

```yaml
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
    publishLocation: 'Container'
```

این کار باعث می‌شود:
- Azure DevOps فایل‌های staging را جمع‌آوری کند
- آن‌ها را در فضای ابری ذخیره کند
- به artifact یک **نام** بدهد (مثلاً `drop`)

> توجه: اگر artifact منتشر نشود، پس از اتمام job فایل‌ها حذف می‌شوند.

---

## 🚀 4. کاربرد Artifact

Artifact معمولاً در مرحله‌ی بعدی **CD (Continuous Deployment)** استفاده می‌شود.  
مراحل معمولی به این صورت است:

```
Build → Artifact (drop) → Deploy
```

مزایای این روش:

- تضمین می‌کند هر deploy از **همان خروجی تست‌شده** انجام شود ✅
- امکان نگهداری نسخه‌های build برای بازگشت به نسخه‌های قبلی ✅
- جداسازی محیط‌های توسعه، تست و production ✅

---

## 📌 5. جمع‌بندی

Artifact همان خروجی قابل استفاده‌ی Build است و در Azure DevOps:

- با `dotnet publish` در مسیر staging ساخته می‌شود
- با `PublishBuildArtifacts` منتشر می‌شود
- در مراحل بعدی deployment استفاده می‌شود

با این روش، فرآیند CI/CD **قابل تکرار، ایمن و قابل کنترل** خواهد بود.

---

## 📊 دیاگرام روند (اختیاری)

```
Source Code → Build → Artifact (drop) → Deploy → Production
```

این نشان می‌دهد که Artifact همان بسته‌ی میانی است که از Build به Deployment منتقل می‌شود.

| محل ذخیره | سرور Azure DevOps |
| دستور انتشار | `PublishBuildArtifacts@1` |
| مشاهده | بخش **Artifacts** در صفحه‌ی Pipeline Run |
| استفاده مجدد | با `download: current` در مراحل بعدی |
| مزیت | جداسازی Build و Deploy، پایداری و تکرارپذیری فرایندها |

