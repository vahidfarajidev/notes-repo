# فهم Artifact در Azure DevOps

در دنیای **CI/CD**، مفهوم **Artifact** نقش کلیدی دارد. این مقاله به زبان ساده توضیح می‌دهد Artifact چیست، چرا مهم است و چطور در Azure DevOps استفاده می‌شود.

---

## 🧱 1. Artifact چیست؟

Artifact در واقع همان **خروجی نهایی مرحله‌ی Build** است که برای مراحل بعدی (مثل Deploy) قابل استفاده است.  
به طور مثال در پروژه‌های .NET خروجی‌ها می‌توانند شامل موارد زیر باشند:

- فایل‌های DLL و EXE
- فایل‌های publish-ready برای deployment
- فایل‌های پیکربندی (appsettings.json و غیره)
- هر فایل دیگری که برای اجرای اپلیکیشن لازم است

> به بیان ساده: Artifact همان "بسته‌ی آماده برای ارسال" است.

---

## 🧩 2. مسیر Artifact Staging Directory

در Azure DevOps، pipeline برای ذخیره‌سازی موقت خروجی build از یک مسیر پیش‌فرض استفاده می‌کند:

```
$(Build.ArtifactStagingDirectory)
```

این مسیر روی ماشین build agent ساخته می‌شود و فایل‌های نهایی build در آن قرار می‌گیرند.

مثال در یک pipeline .NET:

```yaml
- script: dotnet publish --configuration Release --no-build -o $(Build.ArtifactStagingDirectory)
  displayName: 'dotnet publish'
```

این دستور پروژه را **publish** می‌کند و خروجی نهایی را در پوشه staging قرار می‌دهد.

---

## 📦 3. انتشار Artifact

پس از build و publish، فایل‌ها باید برای مراحل بعدی نگهداری شوند. در Azure DevOps این کار با **PublishBuildArtifacts** انجام می‌شود:

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

