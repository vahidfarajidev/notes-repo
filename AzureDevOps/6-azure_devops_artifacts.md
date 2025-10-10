# 🎯 همه‌چیز درباره‌ی Artifact در Azure DevOps

## مقدمه
در فرآیند **CI/CD** با Azure DevOps، زمانی که پروژه‌ی شما Build می‌شود، خروجی این مرحله به عنوان **Artifact** شناخته می‌شود.  
Artifact در واقع مجموعه‌ای از فایل‌ها (مثل DLLها، فایل‌های منتشرشده‌ی وب‌اپلیکیشن، یا بسته‌های Zip) است که در مراحل بعدی Pipeline مثل Deploy یا Test مورد استفاده قرار می‌گیرند.

---

## 🧩 Artifact چیست؟
**Artifact** خروجی نهایی مرحله‌ی Build است که می‌خواهیم برای مراحل بعدی مثل تست، استقرار (Deploy) یا انتشار نگه داریم.

به زبان ساده‌تر:
> Artifact = نتیجه‌ی قابل‌استفاده‌ی Pipeline (مثل خروجی `dotnet publish`)

---

## ⚙️ چرا Artifact مهم است؟
- باعث **جداسازی مراحل Build و Deploy** می‌شود.  
- تضمین می‌کند که در مرحله‌ی Deploy دقیقاً همان خروجی Build قبلی منتشر شود.  
- امکان **دانلود خروجی Build** برای بررسی یا استفاده‌ی دستی را فراهم می‌کند.  
- در محیط‌های چندمرحله‌ای (multi-stage pipelines) به اشتراک‌گذاری خروجی بین مراحل را ممکن می‌سازد.

---

## 🧱 مسیرهای مهم در Azure Pipeline

| متغیر | توضیح |
|--------|--------|
| `$(Build.SourcesDirectory)` | مسیر کلون شدن سورس‌کد |
| `$(Build.ArtifactStagingDirectory)` | محل موقت جمع‌آوری فایل‌های نهایی قبل از انتشار |
| `$(Build.BinariesDirectory)` | مسیر فایل‌های کامپایل‌شده |
| `$(Pipeline.Workspace)` | مسیر Workspace فعلی Pipeline |

> ⚠️ توجه: این مسیرها روی Agent موقتی هستند و پس از پایان Pipeline حذف می‌شوند.  
> بنابراین باید خروجی را با دستور `PublishBuildArtifacts@1` به Azure DevOps منتقل کنید.

---

## 🚀 ساخت Artifact در YAML

در فایل `azure-pipelines.yml`، معمولاً Artifact به شکل زیر ساخته می‌شود:

```yaml
pool:
  vmImage: 'ubuntu-latest'

steps:
- task: DotNetCoreCLI@2
  inputs:
    command: 'publish'
    publishWebProjects: true
    arguments: '--configuration Release --output $(Build.ArtifactStagingDirectory)'

- task: PublishBuildArtifacts@1
  inputs:
    pathToPublish: '$(Build.ArtifactStagingDirectory)'
    artifactName: 'drop'
```

در این مثال:
- پروژه با دستور `dotnet publish` ساخته و در مسیر `$(Build.ArtifactStagingDirectory)` قرار می‌گیرد.
- سپس با تسک `PublishBuildArtifacts@1`، خروجی در سرور Azure DevOps ذخیره می‌شود و با نام **drop** شناخته می‌شود.

---

## 🗂 مشاهده و دانلود Artifact

بعد از اجرای Pipeline:
1. وارد **Azure DevOps → Pipelines → Runs** شوید.  
2. روی اجرای مورد نظر کلیک کنید.  
3. در سمت راست (یا پایین صفحه) بخشی به نام **Artifacts** را مشاهده می‌کنید.  
4. با کلیک روی آن می‌توانید:
   - محتویات Artifact را ببینید.
   - یا فایل ZIP را دانلود کنید.

---

## 🔄 استفاده از Artifact در مرحله‌ی بعدی (Deploy)

اگر Pipeline چند مرحله دارد، می‌توانید Artifact ساخته‌شده را در Stage بعدی دانلود کنید:

```yaml
- download: current
  artifact: drop

- task: AzureWebApp@1
  inputs:
    appName: 'MyWebApiApp'
    package: '$(Pipeline.Workspace)/drop'
```

در این حالت:
- فایل Artifact از مرحله‌ی قبلی گرفته می‌شود.
- سپس از آن برای Deploy در Azure App Service استفاده می‌شود.

---

## 🧹 نگه‌داری و حذف Artifactها
Azure DevOps به‌صورت پیش‌فرض Artifactها را تا مدتی (مثلاً 30 روز) نگه می‌دارد.  
می‌توانید در بخش تنظیمات پروژه → **Retention policies** مشخص کنید که چه مدت نگه داشته شوند.

---

## 💡 نکات مفید
- نام Artifact را واضح و توصیفی بگذارید (مثلاً `webapi-build` یا `frontend-dist`).
- اگر حجم Artifact زیاد است، فقط فایل‌های لازم را کپی یا فشرده کنید.
- می‌توانید از چند `PublishBuildArtifacts` برای تولید چند خروجی مختلف استفاده کنید.
- در پروژه‌های بزرگ‌تر، از **Pipeline Artifacts** (به‌جای Build Artifacts سنتی) استفاده کنید — سریع‌تر و کارآمدترند.

---

## 🧭 جمع‌بندی

| مفهوم | توضیح |
|--------|--------|
| Artifact | خروجی نهایی مرحله‌ی Build |
| محل ذخیره | سرور Azure DevOps |
| دستور انتشار | `PublishBuildArtifacts@1` |
| مشاهده | بخش **Artifacts** در صفحه‌ی Pipeline Run |
| استفاده مجدد | با `download: current` در مراحل بعدی |
| مزیت | جداسازی Build و Deploy، پایداری و تکرارپذیری فرایندها |

---

### 📘 منابع پیشنهادی
- [Microsoft Docs – Publish and Download Artifacts](https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/pipeline-artifacts)
- [YAML Schema Reference](https://learn.microsoft.com/en-us/azure/devops/pipelines/yaml-schema)
- [Azure Pipelines Agents and Pools](https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/agents)

---

_نویسنده: شما ✨_  
_تاریخ: 2025-10-10_

