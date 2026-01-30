# ساخت پروژه HTML/CSS/JS/jQuery در GitLab با روش Git محلی و HTTPS

این مقاله قدم‌به‌قدم نشان می‌دهد چگونه یک پروژه جدید در GitLab ایجاد کنیم، از طریق کامپیوتر محلی فایل‌ها را Push کنیم و یک نفر جدید را فقط به همین پروژه اضافه کنیم. سناریو شامل سه Owner در گروه، نفر جدید محدود به یک پروژه و پروژه Private است.

---

## 1️⃣ ساخت پروژه در GitLab

1. وارد **Group (Lotuscode)** شو
2. روی **New Project → Create blank project** کلیک کن
3. فرم را پر کن:
   - **Project name**: SampleProject (یا هر اسم دلخواه)
   - **Visibility**: **Private** ✅
   - Deployment Target: **هیچ‌کدام** انتخاب نکن (اختیاری)
4. **Project Configuration**:
   - **Initialize repository with README** → ❌ بردار (چون از Local Push استفاده می‌کنیم)
   - **SAST** → ❌ غیرفعال
   - **Secret Detection** → ❌ غیرفعال
5. روی **Create Project** کلیک کن ✅

> پروژه Private ساخته شد، Ownerهای گروه همه چیز را می‌بینند، نفر جدید هنوز اضافه نشده.

---

## 2️⃣ آماده کردن پروژه روی کامپیوتر (Local)

1. پوشه پروژه بساز:
```bash
mkdir SampleProject
cd SampleProject
```
2. فایل‌های پروژه HTML/CSS/JS/jQuery بساز:
```
SampleProject/
│── index.html
│── style.css
│── script.js
│── jquery.min.js (یا از CDN استفاده کن)
```

---

## 3️⃣ مقداردهی اولیه Git

```bash
git init --initial-branch=main
```
* شاخه اصلی پروژه main می‌شود.

---

## 4️⃣ اتصال به GitLab با HTTPS

```bash
git remote add origin https://gitlab.com/lotuscode/gsmproject.git
```
> چون URL پروژه شما HTTPS است، این روش راحت و امن است.

---

## 5️⃣ اضافه کردن فایل‌ها و Commit اولیه

```bash
git add .
git commit -m "Initial commit - HTML/CSS/JS/jQuery project"
```

---

## 6️⃣ Push پروژه به GitLab

```bash
git push --set-upstream origin main
```
* پروژه روی GitLab Private ایجاد شد و فایل‌ها بالا رفت.

---

## 7️⃣ اضافه کردن نفر جدید فقط به این پروژه

1. داخل پروژه GitLab برو → **Project → Members**
2. روی **Invite members** کلیک کن
3. ایمیل یا Username نفر جدید را وارد کن
4. Role بده: **Developer** (یا Reporter، بسته به نیاز)
5. Confirm → نفر جدید **فقط همین پروژه را می‌بیند**

> Ownerهای گروه همچنان همه پروژه‌ها و نفر جدید را می‌بینند.

---

## 8️⃣ نکات تکمیلی

* اگر بعداً خواستی پروژه آنلاین شود → می‌توانی **GitLab Pages** یا **Web Deployment Platform** فعال کنی
* نفر جدید دیگر پروژه‌های گروه را نمی‌بیند، چون پروژه **Private** است
* هیچ Notification برای Ownerها ارسال نمی‌شود وقتی پروژه ساخته شد

---

💡 **چک‌لیست نهایی برای سناریوی شما:**

1. ایجاد پروژه Private بدون README
2. ساخت پوشه و فایل‌ها روی Local
3. `git init --initial-branch=main`
4. اتصال HTTPS به GitLab (`git remote add origin ...`)
5. `git add .` و `git commit`
6. `git push --set-upstream origin main`
7. اضافه کردن نفر جدید فقط به پروژه → Developer
8. بررسی دسترسی‌ها → نفر جدید فقط پروژه جدید، Ownerها همه چیز


---

## 9️⃣ نکته مهم درباره `git push`

دستور زیر فقط **بار اول** لازم است اجرا شود:

```bash
git push --set-upstream origin main
```

این دستور:
- شاخه‌ی `main` را روی GitLab می‌سازد
- ارتباط بین شاخه‌ی محلی `main` و شاخه‌ی ریموت `origin/main` را تنظیم می‌کند

### ✅ از این به بعد (روال روزمره)

بعد از هر تغییر در پروژه، این دستورات کافی هستند:

```bash
git status
git add .
git commit -m "your message"
git push
```

Git از این به بعد می‌داند که کدها را به کدام Repository و Branch ارسال کند.

### ⚠️ چه زمانی دوباره `--set-upstream` لازم می‌شود؟

فقط زمانی که **Branch جدید** بسازید، مثلاً:

```bash
git checkout -b feature/new-ui
git push --set-upstream origin feature/new-ui
```

---

✅ **جمع‌بندی:**
- اولین Push → `git push --set-upstream origin main`
- Pushهای بعدی → فقط `git push`

