# 🚀 مرحله ۱: ساخت Azure Container Registry (ACR) و Push کردن Docker Image

در این مرحله یاد می‌گیریم چطور Docker image پروژه‌ی خودمان را از سیستم محلی به **Azure Container Registry (ACR)** ارسال کنیم تا بعداً بتوانیم آن را از طریق Azure Pipeline یا Kubernetes اجرا کنیم.

---

### ۱) ساخت ACR (از طریق Azure CLI)

> اگر از Azure CLI لاگین نیستی، ابتدا دستور زیر را بزن:
>
> ```bash
> az login
> ```

حالا با دستورات زیر ACR بساز:

در واقع Resource Group یک کانتینر منطقی برای منابع Azure است (مثل VM، Storage، ACR و غیره).

و ACR_NAME اسم یکتا برای Azure Container Registry (ACR).

همچنین azuredemoacr$(date +%s | tail -c 4) یعنی azuredemoacr + آخرین ۴ رقم timestamp فعلی، تا اسم یکتا باشد. چون Azure قوانین دارد: حروف کوچک و بدون فاصله باید باشد.

و LOCATION منطقه جغرافیایی دیتاسنتر Azure، مثل westeurope، که Resource Group و ACR در آن ایجاد می‌شوند.



```bash
# متغیرها — اینها رو طبق نیازت تغییر بده
RESOURCE_GROUP="AzureDemo-RG"
ACR_NAME="azuredemoacr$(date +%s | tail -c 4)"  # مثال: azuredemoacr1234 (اسم باید یکتا و با حروف کوچک باشد)
LOCATION="westeurope"

# ساخت Resource Group (اگر قبلاً ساخته شده نگهش دار)
az group create --name $RESOURCE_GROUP --location $LOCATION

# ساخت ACR (Basic یا Standard)
az acr create --resource-group $RESOURCE_GROUP --name $ACR_NAME --sku Standard --location $LOCATION
```

📘 `az acr create` خروجی‌ای می‌دهد که نام ACR را تأیید می‌کند.
نام نهایی ACR را ذخیره کن (مثلاً `azuredemoacr1234`).

---

### ۲) لاگین Docker به ACR

```bash
az acr login --name <ACR_NAME>
# مثال:
# az acr login --name azuredemoacr1234
```

این دستور Docker را به رجیستری Azure لاگین می‌کند (نیاز به `docker login` جدا نیست اگر `az acr login` اجرا شود).

---

### ۳) تگ کردن image و Push

فرض می‌کنیم image لوکالت اسمش `azuredemoapi:latest` است. حالا آن را تگ کرده و به ACR ارسال می‌کنیم:

```bash
LOCAL_IMAGE="azuredemoapi:latest"
ACR_NAME="<ACR_NAME>"                    # جایگزین کن
ACR_LOGIN_SERVER="$(az acr show --name $ACR_NAME --query loginServer --output tsv)"

# تگ کردن
docker tag $LOCAL_IMAGE $ACR_LOGIN_SERVER/azuredemoapi:1.0.0

# Push
docker push $ACR_LOGIN_SERVER/azuredemoapi:1.0.0
```

📦 خروجی `docker push` باید نشان دهد که لایه‌ها آپلود شده‌اند و در نهایت image در ACR موجود است.

---

### ۴) بررسی image داخل ACR

```bash
az acr repository list --name $ACR_NAME --output table
az acr repository show-tags --name $ACR_NAME --repository azuredemoapi --output table
```

این دستورات نشان می‌دهند که repository و تگ‌ها در ACR ذخیره شده‌اند.

---

### 💡 نکات سریع

* نام Image داخل ACR به شکل `<loginServer>/<repo>:<tag>` است؛ `loginServer` مثل `azuredemoacr1234.azurecr.io`.
* از تگ معنایی استفاده کن (مثلاً `1.0.0`, `ci-<buildId>`) تا versioning و rollback ساده‌تر شود.
* بعداً در Pipeline می‌توانیم مرحله‌ای اضافه کنیم که به‌صورت خودکار image بسازد و به همین ACR push کند (در قدم بعدی انجام می‌دهیم).

---

✅ اگر آماده‌ای برای مرحله‌ی بعد (تنظیم Pipeline برای Build و Push خودکار)، فقط بگو:

> **برو قدم بعدی**

