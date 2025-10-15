در yml فایل:
variables:
  dockerRegistryServiceConnection: 'AzureDemo-Connection'  # Service Connection به Azure
  imageRepository: 'azuredemoapi'
  containerRegistry: '<ACR_NAME>' # نام ACR
  tag: '1.0.0'
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

