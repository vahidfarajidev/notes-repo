# توضیح فایل azure-pipelines.yml

در حین اینکه Azure DevOps فایل YAML را می‌خواند و Pipeline را ایجاد می‌کند، بد نیست یک توضیح کامل از **نقش و ساختار فایل YAML در CI/CD** داشته باشیم.

---

## ۱️⃣ هدف فایل

- این فایل به Azure DevOps می‌گوید که **چطور پروژه را Build، Test و Publish کند**.
- هر کاری که در Pipeline لازم است (نصب SDK، بازیابی پکیج‌ها، اجرای تست‌ها، تولید Artifact) در این فایل تعریف می‌شود.
- با استفاده از YAML، Pipeline **Version Control Friendly** می‌شود و می‌تواند داخل ریپو نگهداری شود.

---

## ۲️⃣ بخش‌های اصلی فایل

### 2.1 Trigger
```yaml
trigger:
  branches:
    include:
      - main
```
- مشخص می‌کند Pipeline **چه زمانی اجرا شود**.
- در این مثال، هر `push` یا `merge` به شاخه `main` Pipeline را فعال می‌کند. یعنی هر بار که در شاخه main تغییری Push کنی (کد جدید، تغییر فایل، حتی Commit خالی)، Azure DevOps به صورت خودکار Pipeline رو Run می‌کنه. نیازی نیست هیچ دکمه‌ای بزنی.

### 2.2 Pool
```yaml
pool:
  vmImage: 'ubuntu-latest'
```
- تعریف می‌کند که **Agent روی چه سیستم عاملی اجرا شود**.
- یعنی: "Azure DevOps لطفاً برای اجرای این Pipeline از یک ماشین مجازی (Virtual Machine) با سیستم‌عامل Ubuntu (آخرین نسخه‌ی پایدار) استفاده کن."
- این ماشین مجازی در سرورهای Microsoft Azure اجرا می‌شود. یعنی لازم نیست خودت سروری راه‌اندازی کنی — Azure DevOps خودش هنگام اجرای Pipeline، به‌صورت خودکار یک VM (ماشین مجازی موقتی) ایجاد می‌کند، دستورات تو را اجرا می‌کند، و بعد از پایان کار آن را پاک می‌کند.
- `ubuntu-latest` یک ماشین مجازی لینوکس است که تمام ابزارهای لازم برای .NET نصب شده دارند.
- کلمه‌ی pool تعیین می‌کند که Pipeline روی چه Agent‌ای اجرا شود. در DevOps، وقتی یک Pipeline اجرا می‌کنی، دستورهای YAML (مثل build، test و publish) باید روی یک سیستم واقعی یا مجازی اجرا شوند. این سیستم را Build Agent می‌گویند.

### 2.3 Variables
```yaml
variables:
  buildConfiguration: 'Release'
  dotnetVersion: '9.0.x'
```
- متغیرها کمک می‌کنند **مقدارها را یکجا تغییر دهی** و در چند مرحله استفاده شوند.
- مثال: Configuration پروژه (`Release` یا `Debug`) و نسخه SDK دات‌نت.

### 2.4 Steps
- شامل **دستورات واقعی Build و Test و Publish** است.
- می‌تواند `script` (دستور خط فرمان) یا `task` (یک Task آماده Azure DevOps) باشد.

مثال‌ها در فایل ما:
```yaml
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '$(dotnetVersion)'

- script: dotnet restore
- script: dotnet build --configuration $(buildConfiguration)
- script: dotnet test --logger "trx;LogFileName=test_results.trx"
- script: dotnet publish -o $(Build.ArtifactStagingDirectory)
- task: PublishBuildArtifacts@1
```
- `UseDotNet@2` → نصب نسخه مشخص SDK  
- `dotnet restore` → بازیابی پکیج‌ها  
- `dotnet build` → ساخت پروژه  
- `dotnet test` → اجرای تست‌های واحد  
- `dotnet publish` → خروجی برای استقرار تولید می‌کند  
- `PublishBuildArtifacts@1` → انتشار خروجی به عنوان Artifact قابل دانلود یا Deploy

---

## ۳️⃣ مزایای استفاده از YAML

- **Versioned**: داخل Git نگهداری می‌شود  
- **Portable**: می‌توانی به راحتی Pipeline را بین پروژه‌ها منتقل کنی  
- **Readable**: خواندن و فهم Pipeline ساده است، مخصوصاً وقتی با کامنت‌ها توضیح داده شود  
- **Flexible**: می‌توان چندین Stage، Job و Task تعریف کرد

---

💡 خلاصه:  
فایل `azure-pipelines.yml` در واقع **نقشه راه CI/CD پروژه توست**. Azure DevOps بر اساس این نقشه پروژه را Build، Test و Artifact تولید می‌کند، و هر Push به شاخه `main` باعث اجرای خودکار Pipeline می‌شود.

