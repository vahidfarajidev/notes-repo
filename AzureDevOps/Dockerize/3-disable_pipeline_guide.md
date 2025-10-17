# غیرفعال کردن اجرای اتوماتیک Pipeline در Azure DevOps

گاهی اوقات نیاز داریم که اجرای اتوماتیک یک Pipeline در Azure DevOps را متوقف کنیم، مخصوصاً وقتی چند Pipeline مشابه داریم یا در حال ایجاد یک Pipeline جدید برای Docker هستیم.

---

## 🔹 مراحل غیرفعال کردن Pipeline

1. وارد Azure DevOps شو و پروژه خود را باز کن.
2. به مسیر `Pipelines > Pipelines` برو.
3. Pipeline مورد نظر را پیدا کن (مثلاً pipeline قدیمی که بدون Docker است).
4. روی سه نقطه (•••) کنار نام Pipeline کلیک کن.
5. گزینه **Pause/Stop** یا **Disable** را انتخاب کن.

> ⚠️ اگر گزینه Disable را نمی‌بینی، ممکن است نوع Pipeline YAML باشد که مستقیماً با فایل `azure-pipelines.yml` در ریپو مرتبط است. در این صورت:
> - می‌توانی trigger فایل YAML را موقتا حذف یا کامنت کنی:
>
> ```yaml
> trigger:
>   branches:
>     include: [] # خالی یعنی هیچ برنچی اتوماتیک تریگر نمی‌شود
> ```
> - یا فایل YAML را به شاخه دیگری منتقل یا تغییر نام بده.

---

## 🔹 نکته PR

همچنین Pipeline‌ها می‌توانند هم برای Push اتوماتیک و هم برای Pull Request (PR) تریگر شوند. حتی اگر در حال حاضر PR تریگری نداشته باشی، ممکن است بعداً اضافه شود. برای اطمینان:

- بررسی کن که در فایل YAML بخش زیر وجود ندارد یا خالی باشد:

```yaml
pr:
  branches:
    include: []
```

- اگر این بخش خالی باشد، هیچ PR باعث اجرای Pipeline نخواهد شد.

---
## ارتباط CI و CD در Azure DevOps

Pipeline در Azure DevOps معمولاً به دو بخش تقسیم می‌شود:

## 1. CI (Continuous Integration)
- Build و Test پروژه
- وقتی Push یا PR اتفاق می‌افتد، اجرا می‌شود

## 2. CD (Continuous Deployment)
- Deploy خروجی CI به محیط مثل Azure App Service یا Container
- معمولا وابسته به Build موفق CI است

## نکات مهم وقتی Trigger غیرفعال باشد
- هیچ Push یا PR جدیدی باعث اجرای CI نمی‌شود
- بدون CI جدید، CD هم اجرا نخواهد شد چون Artifact یا Image جدیدی برای Deploy ساخته نشده است

> 🔹 نکته: اگر Pipeline را دستی Run کنی، CI و CD هر دو اجرا می‌شوند، چون Run دستی کل Pipeline را Trigger می‌کند.
---

## ✅ جمع‌بندی

- برای غیرفعال کردن اجرای اتوماتیک، می‌توان از گزینه Disable در UI استفاده کرد.
- اگر YAML است، trigger و pr را خالی یا کامنت کن.
- این روش تضمین می‌کند که Pipeline قدیمی اجرا نشود و تداخل با Pipeline جدید Docker ایجاد نکند.
-  توجه مهم: پایپلاین روی Push اتوماتیک به شاخه main اجرا نخواهد شد. به عبارتی، فقط وقتی دستی روی Run pipeline در Azure DevOps کلیک کنی، اجرا می‌شود. برای اطمینان، بعد از Commit و Push، می‌تونی در Azure DevOps → Pipelines → Your pipeline بررسی کنی که هیچ Run اتوماتیک جدیدی ایجاد نمی‌شود.
-  می‌تونی یک کامیت خالی (empty commit) انجام بدی تا فقط Triggerها و تغییرات فایل YAML رو تست کنی بدون اینکه کد دیگری تغییر کنه:

```bash
git commit --allow-empty -m "Test pipeline trigger after disabling triggers"
git push origin main
```

این کار باعث می‌شود یک Commit جدید روی برنچ `main` ساخته شود و با بررسی Pipeline می‌توانی ببینی که آیا CI و CD اجرا می‌شوند یا خیر. چون Triggerها غیرفعال هستند، باید ببینی که Pipeline به صورت خودکار اجرا نمی‌شود.

---
## نکته مهم: با انجام موارد بالا هم Disable انجام نشد لذا مرحله ی زیر هم لازم است:

## راهنمای غیر فعال کردن اجرای اتوماتیک Pipeline در Azure DevOps

این یک **راهنمای کامل و مرحله‌به‌مرحله** برای اینکه pipeline YAML شما روی push اتوماتیک اجرا نشود، فقط از بخش **Settings / Triggers** و بدون تغییر دیگر pipelineها است.

---

## 🔹 مراحل غیر فعال کردن اجرای اتوماتیک pipeline

1. **وارد Azure DevOps → پروژه خودت → Pipelines** شو.
2. روی **pipeline مورد نظر** (مثلاً `AzureDemoApi`) کلیک کن.
3. روی دکمه Edit کلیک کن.
4. از بالای صفحه روی **… (سه نقطه)** کلیک کن و گزینه **Triggers** رو انتخاب کن.
5. در بخش **Continuous Integration (CI)**:
   - تیک **Disable continuous integration** را بزن.
6. **Save** را بزن.

---

## 🔹 چه اتفاقی می‌افتد

- **CI اتوماتیک غیرفعال می‌شود** → pipeline دیگر با هر push اجرا نمی‌شود.
- **تنها pipeline فعلی** تحت تاثیر قرار می‌گیرد. سایر pipelineها دست نخورده باقی می‌مانند.
- اجرای دستی pipeline هنوز امکان‌پذیر است (با گزینه **Run pipeline**).
- اگر بعداً بخواهی CI اتوماتیک را فعال کنی، دوباره همین مسیر را برو و گزینه را **Enable** کن.

---

## 🔹 نکته جانبی

- اگر YAML شما هنوز شامل بخش `trigger:` است، با این تنظیم CI اتوماتیک **کاملاً نادیده گرفته می‌شود**.
- این روش **بهترین و امن‌ترین راه** برای جلوگیری از اجرای ناخواسته pipeline بدون حذف یا کامنت کردن کل YAML است.

---




