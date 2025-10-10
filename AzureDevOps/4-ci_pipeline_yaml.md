# مرحله بعد — ساخت CI Pipeline

عالی — مرحله بعد (ساخت **CI pipeline**) رو شروع می‌کنیم، فقط همین قدم رو الان انجام می‌دیم و بعد منتظر فرمان تو برای رفتن قدم بعدی می‌مونم.

---

### 🎯 هدف این قدم

ایجاد یک **Azure Pipelines (YAML)** که با هر `push` به شاخه `main` پروژه‌رو بسازد، تست کنه و Artifact تولید کنه.

---

### ۱) فایل YAML برای Pipeline بساز

در ریشه‌ی پروژه‌ات یک فایل بساز به اسم **`azure-pipelines.yml`** و این محتوا را داخلش قرار بده:

```yaml
# azure-pipelines.yml
# ===================
# CI/CD Pipeline برای پروژه‌های .NET 9
# ساخت، تست و پابلیش اتوماتیک روی شاخه main

trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'ubuntu-latest'

variables:
  buildConfiguration: 'Release'
  dotnetVersion: '9.0.x'

steps:
  # 1️⃣ نصب SDK دات‌نت
  - task: UseDotNet@2
    displayName: 'Install .NET SDK $(dotnetVersion)'
    inputs:
      packageType: 'sdk'
      version: '$(dotnetVersion)'

  # 2️⃣ بازیابی پکیج‌ها
  - script: dotnet restore
    displayName: 'Restore NuGet Packages'

  # 3️⃣ بیلد پروژه
  - script: dotnet build --configuration $(buildConfiguration) --no-restore
    displayName: 'Build Project'

  # 4️⃣ اجرای تست‌ها
  - script: dotnet test --configuration $(buildConfiguration) --no-build --logger "trx;LogFileName=test_results.trx"
    displayName: 'Run Unit Tests'

  # 5️⃣ پابلیش پروژه
  - script: dotnet publish --configuration $(buildConfiguration) --no-build -o $(Build.ArtifactStagingDirectory)
    displayName: 'Publish Project'

  # 6️⃣ انتشار خروجی Build به عنوان Artifacts
  - task: PublishBuildArtifacts@1
    displayName: 'Publish Build Artifacts'
    inputs:
      PathtoPublish: '$(Build.ArtifactStagingDirectory)'
      ArtifactName: 'drop'
      publishLocation: 'Container'
```

---

### ۲) فایل را به Git اضافه و Push کن

در پوشه پروژه اجرا کن:

```bash
git add azure-pipelines.yml
git commit -m "Add Azure Pipelines YAML for CI (build/test/publish artifacts)"
git push origin main
```

---

### ۳) ایجاد Pipeline در Azure DevOps (یک‌بار لازم است)

1. وارد پروژه‌ات در Azure DevOps بشو (`Pipelines` → `New pipeline`)
2. انتخاب کن: **Azure Repos Git** (یا محل ریپوی خودت)
3. انتخاب ریپو: `AzureDemoApi`
4. در مرحله انتخاب، گزینه‌ی **Existing Azure Pipelines YAML file** را بزن و `azure-pipelines.yml` را انتخاب کن.
5. Create / Run را بزن تا pipeline اجرا شود.

> نکته: بعد از این، هر `push` به `main` این pipeline را تریگر می‌کند و خروجی (Artifact) را در بخش Pipelines → Runs می‌بینی.

---

### چه چیزهایی الان آماده شد

* فایل YAML داخل ریپو هست و Push شده ✅
* Pipeline در Azure DevOps قابل ایجاد/اجرای مستقیم هست ✅

---

حالا من منتظر دستور تو هستم:
وقتی گفتی **«برو بعدی»**، قدم بعدی (تنظیمات بخش CD / Deploy به Azure App Service و Service Connection) رو شروع می‌کنم.

