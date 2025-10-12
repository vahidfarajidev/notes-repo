# 🚀 مرحله‌ی Deployment در Azure Pipeline

الان بریم فقط **یک قدم بعدی** — یعنی **تنظیم مرحله‌ی Deployment در Pipeline** تا فایل‌های build شده به App Service که ساختی، به‌صورت خودکار منتقل بشن.

---

## 🔹 مرحله ۶ — اضافه کردن مرحله‌ی Deploy به Azure App Service در YAML Pipeline

### 🎯 هدف این قدم

اتصال مرحله‌ی **Build و Publish** به مرحله‌ی **Deploy** که خروجی (Artifact) روی App Service آپلود شود.

---

### گام ۶.۱ — ویرایش فایل `azure-pipelines.yml`

در انتهای فایل YAML (بعد از مرحله‌ی Publish و `PublishBuildArtifacts`)، این بخش را اضافه کن:

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'AzureDemo-Connection'
    appName: 'AzureDemo'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'
  displayName: 'Deploy to Azure App Service'
```

🔸 توضیحات:

* `azureSubscription`: همون نام Service Connection که مرحله قبل ساختی  
* `appName`: اسم App Service که در Azure Portal ایجاد کردی (`AzureDemo`)  
* `package`: محل خروجی فایل‌های publish شده از مرحله build  

---

### گام ۶.۲ — Commit و Push

بعد از ویرایش و ذخیره فایل، در ترمینال اجرا کن:

```bash
git add azure-pipelines.yml
git commit -m "Add deployment stage to Azure App Service"
git push origin main
```

> ⚡ به‌محض Push، Azure DevOps Pipeline به‌صورت خودکار اجرا می‌شه و در انتها باید مرحله‌ی  
> **“Deploy to Azure App Service”**  
> رو ببینی که وضعیتش ✅ سبز می‌شه.

---

قدم بعدی: **تست نهایی در Azure (بررسی اجرای API آنلاین)**.

