# تفاوت Task و Container در Azure DevOps

در Azure DevOps Pipelines، مفاهیم **Task** و **Container** اغلب با هم استفاده می‌شوند اما معنای متفاوتی دارند.

---

## 1. Task چیست؟

یک `task` در Pipeline یعنی یک **واحد کاری آماده** که می‌تواند کاری انجام دهد. مثال‌ها:

- ساخت پروژه (`Build`)
- اجرای تست‌ها (`Test`)
- انتشار به محیط (`Deploy`)

مثال YAML:
```yaml
task: AzureWebAppContainer@2
displayName: Deploy to Azure App Service (Docker)
```

- `AzureWebAppContainer@2` یک task آماده برای انتشار **کانتینر روی Azure App Service** است.
- `displayName` فقط یک نام خوانا برای Task است.

> پس Task یعنی یک دستورالعمل یا کاری که Pipeline اجرا می‌کند.

---

## 2. Container چیست؟

در همان Task، پارامتر `containers` داریم:
```yaml
containers: '$(containerRegistry)/$(imageRepository):$(tag)'
```

- منظور از **container** همان **Docker image** است که می‌خواهید روی App Service اجرا کنید.
- فرمت استاندارد Docker:
```
[Registry]/[Repository]:[Tag]
```
مثال:
```
myregistry.azurecr.io/myapp:1.0.0
```

> پس container یعنی بسته نرم‌افزاری آماده شامل اپلیکیشن و وابستگی‌ها که Task آن را مستقر می‌کند.

---

## 3. رابطه بین Task و Container

- **Task**: می‌گوید «یک کار انجام بده» (مثلاً انتشار)
- **Container**: داده‌ای که Task روی آن عمل می‌کند (Docker image)

تصور ساده:
```
Task = کارگر
Container = چیزی که کارگر با آن کار می‌کند
```

مثال دیگر:
- Task = "نصب برنامه روی سرور"
- Container = "فایلی که شامل برنامه و وابستگی‌هاست"

---

## خلاصه
- Task = دستورالعمل در Pipeline
- Container = بسته نرم‌افزاری آماده برای اجرا
- Task روی Container عمل می‌کند تا اپلیکیشن را در محیط هدف مستقر کند.

