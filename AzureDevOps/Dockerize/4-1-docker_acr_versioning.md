در فایل YAML:
```yaml
...
variables:
  dockerRegistryServiceConnection: 'AzureDemo-Connection'  # Service Connection به Azure
  imageRepository: 'azuredemoapi'
  containerRegistry: '<ACR_NAME>' # نام ACR
  tag: '1.0.0'
...
```
اگر فقط یک تگ ثابت استفاده کنی مثل `1.0.0` یا حتی `latest` و هر بار Pipeline اجرا شود:

* همان Image در ACR **بازنویسی می‌شود** (overwrite).
* نسخه قبلی از تگ همان‌جا حذف یا جایگزین می‌شود، مگر اینکه تگ دیگری داشته باشد.

---

### راهکار درست برای versioning

1. **استفاده از Semantic Versioning یا Build ID**:
   هر بار که Pipeline اجرا می‌شود، یک تگ جدید ایجاد کن. مثال‌ها:

```yaml
tag: '1.0.0'           # نسخه ثابت
tag: '1.0.$(Build.BuildId)'  # Build ID خودکار Azure DevOps
tag: 'ci-$(Build.BuildId)'   # CI-specific tag
```

2. **Pipeline Variables**:
   می‌توانی در Variables یا در Script YAML یک تگ پویا بسازی و استفاده کنی:

```yaml
variables:
  tag: '$(Build.BuildId)'   # هر Build تگ جدید
```

3. **Push با چند تگ همزمان** (اختیاری):
   گاهی هم می‌تونی Image را با `latest` و با تگ Build-specific Push کنی تا هم همیشه آخرین نسخه مشخص باشد و هم History نگه داشته شود.

---

✅ جمع‌بندی:

* اگر تگ ثابت باشد → همیشه overwrite می‌شود
* اگر تگ پویا باشد → نسخه‌های مختلف حفظ می‌شوند و rollback راحت‌تر است
* 
  ---

## مدیریت Docker Image ها در ACR و Azure App Service Free

وقتی روی **Azure App Service Free/Shared** کار می‌کنیم:

## 1. یک Container Image فقط Deploy می‌شود
- App Service فقط آخرین Image که Push و Deploy شده را استفاده می‌کند.
- نسخه‌های قبلی در ACR باقی می‌مانند اما در App Service فقط نسخه آخر فعال است.

## 2. نگه داشتن Image‌های متعدد در ACR
- **ACR** محل نگهداری Image هاست.
- هر تگ جدید Image را جدا نگه می‌دارد و امکان rollback وجود دارد.
- روی App Service Free، فقط یک نسخه Deploy شده فعال است.

## 3. توصیه برای پروژه تستی
- می‌توان از **تگ پویا برای versioning** استفاده کرد (مثلاً `ci-<BuildId>`).
- برای جلوگیری از پر شدن ACR یا هزینه اضافه، Imageهای قدیمی را می‌توان پاک کرد:

```bash
# لیست تگ‌ها
az acr repository show-tags --name <ACR_NAME> --repository azuredemoapi --output table

# حذف تگ قدیمی
az acr repository delete --name <ACR_NAME> --repository azuredemoapi --tag <old-tag> --yes
```

## 💡 جمع‌بندی

- روی **App Service Free**، Deploy همیشه یک Image است → سرور اشغال نمی‌شود.
- در **ACR** نسخه‌های متعدد باقی می‌مانند → امکان rollback وجود دارد.
- برای پروژه تستی → می‌توان چند Build ساخت و بعد تگ‌های قدیمی را پاک کرد تا محیط تمیز بماند.

---

### نکته اختیاری:
می‌توان یک **Pipeline با Cleanup اتوماتیک تگ‌های قدیمی ACR** ساخت تا همیشه محیط مرتب باشد و نسخه‌های قدیمی به صورت خودکار حذف شوند.



