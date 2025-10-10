# راهنمای انتقال پروژه محلی به Azure DevOps

این مقاله توضیح می‌دهد که چگونه یک پروژه محلی Git را به Azure DevOps منتقل کنیم و اولین Push را انجام دهیم.

---

## مراحل انجام کار

### **۱. ساخت پروژه و ریپوزیتوری در Azure DevOps**
- وارد [Azure DevOps](https://dev.azure.com/) شو
- یک پروژه جدید بساز، به عنوان مثال: `<PROJECT_NAME>`
- داخل پروژه، یک ریپوزیتوری بساز، به عنوان مثال: `<REPO_NAME>`

---

### **۲. آماده‌سازی ریپوزیتوری محلی**
- اطمینان از اینکه پروژه محلی Git دارد. اگر ندارد:
```powershell
git init
```
- مطمئن شدن از شاخه اصلی:
```powershell
git branch -M main
```

---

### **۳. اضافه کردن Remote به Azure DevOps**
- URL ریپو را از صفحه Azure DevOps کپی کن و دستور زیر را اجرا کن:
```powershell
git remote add origin https://dev.azure.com/<ORG_NAME>/<PROJECT_NAME>/_git/<REPO_NAME>
```
- اگر قبلاً Remote اضافه شده، ابتدا آن را حذف کن:
```powershell
git remote remove origin
```

---

### **۴. Push اولیه با Force**
- چون ریپوی Remote خالی است و می‌خوای محتوای محلی را جایگزین کنی، از Force استفاده می‌کنیم:
```powershell
git push -u origin main --force
```
- گزینه `-u` باعث می‌شود دفعات بعدی فقط `git push` کافی باشد.

---

### **۵. وضعیت نهایی**
- ریپوی Remote و Local همگام هستند
- شاخه Local و Remote هر دو: `main`
- می‌توانی از این به بعد تغییرات را Commit و Push کنی و سایر اعضای تیم هم می‌توانند Clone یا Pull کنند.

💡 **نکات مهم:**
- اگر ریپوی Remote قبلاً شامل commit است، Force Push همه آن‌ها را پاک می‌کند. تنها زمانی استفاده شود که مطمئن هستی تغییرات محلی باید جایگزین شوند.
- نام شاخه اصلی معمولاً `main` است، اگر محلی `master` باشد، ابتدا آن را تغییر بده.

---

