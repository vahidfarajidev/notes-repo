# Azure DevOps Pipeline برای Build و Push Docker Image به ACR

حالا **یک قدم بعدی** — یعنی **ساخت Pipeline در Azure DevOps برای Build و Push اتوماتیک Docker Image به ACR**.

---

## 🔹 مرحله ۱۱ — ساخت CI/CD Pipeline برای Docker

### 🎯 هدف این قدم

هر بار که کد در ریپو Push می‌شود، Pipeline:

1. Docker Image بسازد
2. Image را به Azure Container Registry (ACR) Push کند

---

### گام ۱ — ایجاد فایل YAML Pipeline

در ریشه پروژه، فایل جدید بساز **`azure-pipelines-docker.yml`** و محتوا را قرار بده:

```yaml
trigger:
  branches:
    include:
      - main

pool:
  vmImage: 'windows-latest'

variables:
  dockerRegistryServiceConnection: 'AzureDemo-Connection'  # Service Connection به Azure
  imageRepository: 'azuredemoapi'
  containerRegistry: '<ACR_NAME>' # نام ACR
  tag: '1.0.0'

steps:
- task: Docker@2
  displayName: Build Docker Image
  inputs:
    command: build
    Dockerfile: '**/Dockerfile'
    tags: |
      $(tag)
    buildContext: .

- task: Docker@2
  displayName: Push Docker Image to ACR
  inputs:
    command: push
    repository: $(containerRegistry)/$(imageRepository)
    tags: |
      $(tag)
    azureSubscription: $(dockerRegistryServiceConnection)
    containerRegistry: $(containerRegistry)
```

🔹 توضیحات:

* `dockerRegistryServiceConnection`: همون Service Connection که به Azure و ACR وصل می‌شود
* `containerRegistry`: اسم ACR
* `imageRepository`: نام ریپوی Image داخل ACR
* `tag`: شماره نسخه Docker Image

---

### گام ۲ — Push و اجرای Pipeline

1. فایل YAML را به Git اضافه و Commit کن:

```powershell
git add azure-pipelines-docker.yml
git commit -m "Add Docker build & push pipeline"
git push origin main
```

2. در Azure DevOps → Pipelines → New pipeline → Azure Repos Git → Existing YAML file → فایل `azure-pipelines-docker.yml` را انتخاب کن و Run

* Pipeline هر Push به `main` را تریگر می‌کند
* Image را Build و Push به ACR انجام می‌دهد

---

✅ تا این مرحله، پروژه **Dockerized** و Pipeline برای **CI/CD Docker Image → ACR** آماده شده است.

---

وقتی گفتی **«برو قدم بعدی»**، می‌ریم سراغ **Deploy اتوماتیک این Docker Image از ACR به Azure App Service یا Azure Container Apps**.

