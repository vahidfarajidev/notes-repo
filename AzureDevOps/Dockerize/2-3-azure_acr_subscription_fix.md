# رفع خطای Subscription برای Azure Container Registry (ACR)

این خطا کاملاً مشخص است: **Subscription تو هنوز برای استفاده از ACR ثبت نشده است**.

خطای اصلی:

```
(MissingSubscriptionRegistration) The subscription is not registered to use namespace 'Microsoft.ContainerRegistry'
```

یعنی سرویس Azure Container Registry (`Microsoft.ContainerRegistry`) هنوز برای Subscription فعلی فعال نشده است.

---

## راه حل

### 1. ثبت namespace مورد نیاز برای Subscription

در Developer PowerShell، دستور زیر را اجرا کن:

```powershell
az provider register --namespace Microsoft.ContainerRegistry
```

- این دستور سرویس ACR را برای Subscription فعال می‌کند.
- معمولاً چند دقیقه طول می‌کشد تا ثبت کامل شود.

### 2. بررسی وضعیت ثبت

بعد از چند ثانیه می‌توانی بررسی کنی که ثبت انجام شده یا نه:

```powershell
az provider show --namespace Microsoft.ContainerRegistry --query "registrationState"
```

- خروجی باید `Registered` باشد.

### 3. اجرای مجدد ساخت ACR

حالا می‌توانی دوباره دستور ساخت ACR را اجرا کنی:

```powershell
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Standard --location $LOCATION
```

✅ اگر همه چیز درست باشد، ACR ساخته خواهد شد.

---

## نکات مهم

- بعضی سرویس‌ها در Azure نیاز دارند که **Subscription برای namespace مربوطه ثبت شود** قبل از اینکه بتوان آن سرویس را ایجاد کرد.
- این ثبت فقط یک بار لازم است و بعداً نیازی به ثبت مجدد نیست.

