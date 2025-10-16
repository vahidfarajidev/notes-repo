نقش **Login Server** توی مبحث **ACR** و **Docker Registry Connection** خیلی مهمه. 👇

---

### 🤩 توی اون متن، Login Server چیه؟

وقتی گفتیم:

> Azure DevOps باید بدونه:
>
> * Login Server کجاست → `azuredemoacr2873.azurecr.io`

منظور از **Login Server** همون **آدرس رجیستری ACR**ه.

در واقع Azure به هر **Azure Container Registry (ACR)** یه آدرس اختصاصی می‌ده که Docker ازش برای login و push/pull استفاده می‌کنه.

---

### 🗍️ به‌طور خلاصه

| مورد | معنی |
|------|------|
| **Login Server** | آدرس اینترنتی رجیستری توی Azure |
| **کاربردش** | آدرسیه که Docker یا Azure DevOps برای اتصال و احراز هویت ازش استفاده می‌کنن |
| **فرمتش** | `<registry-name>.azurecr.io` |
| **مثال** | `azuredemoacr2873.azurecr.io` |

---

### ⚙️ در عمل

وقتی DevOps می‌خواد به ACR وصل بشه، باید بدونه:

```yaml
dockerRegistryServiceConnection: 'AzureDemo-ACR'
```

و در اون Service Connection، Azure DevOps از این مقادیر استفاده می‌کنه:

```
Login Server: azuredemoacr2873.azurecr.io
Username: azuredemoacr2873
Password: <Service Principal Key یا Token>
```

---

### 🔗 جمع‌بندی

> **Login Server = نقطهٔ ورود (Endpoint) برای دسترسی به ACR**  
> مثل URLیه که Docker ازش برای Push و Pull کردن Image‌ها استفاده می‌کنه.

