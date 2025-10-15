# 🚀 Dockerization پروژه HelloController و Pipeline CI/CD

در این مستند، مرحله‌به‌مرحله نحوه Dockerize کردن پروژه **HelloController** و ساخت **Pipeline CI/CD برای Docker** توضیح داده شده است. چون سیستم شما Docker نصب دارد، می‌توانیم همه چیز را لوکال تست کنیم قبل از ارسال به Azure.

---

## 🔹 مرحله ۱ — ساخت Dockerfile

در ریشه پروژه (همان جایی که فایل `.csproj` هست) یک فایل بساز به نام **Dockerfile** و محتوای زیر را قرار بده:

```dockerfile
# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out

# Stage 2: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:9.0
WORKDIR /app
COPY --from=build /app/out ./

EXPOSE 80
ENTRYPOINT ["dotnet", "AzureDemoApi.dll"]
```

### 🔹 توضیح مراحل:

1. **Stage Build**: پروژه را Build و Publish می‌کند
2. **Stage Runtime**: یک Image سبک از ASP.NET Core Runtime می‌سازد و فایل‌های منتشر شده را کپی می‌کند
3. **EXPOSE 80**: پورت HTTP اپلیکیشن
4. **ENTRYPOINT**: اجرای اپلیکیشن

---

## 🔹 مرحله ۲ — Build کردن Docker Image لوکال

در ترمینال پروژه (جایی که Dockerfile هست) اجرا کن (می تونی با Developer PowerShell هم اجرا کنی، ابتدا با CD ... به مسیر پروژه برو، سپس دستور رو اجرا کن):

```bash
docker build -t azuredemoapi:latest .
```

* این دستور یک Image به اسم `azuredemoapi:latest` می‌سازد
* می‌توانی اسم و تگ دلخواه هم بدهی، مثلا `vahid/azuredemoapi:1.0`

  ### توضیح سینتکس:
* `docker build` → دستور ساخت Docker Image
* `-t azuredemoapi:latest` → تعیین **نام و تگ (Tag)** برای Image
  * `azuredemoapi` = نام Image
  * `latest` = تگ Image (می‌توان نام دلخواه دیگری هم گذاشت)
* `.` → مسیر **Docker context**؛ یعنی همه فایل‌ها و فولدرهای مسیر جاری برای Build در دسترس هستند

### نکات:
1. **مکان اجرا:** باید در مسیر فولدری که Dockerfile قرار دارد باشی.
2. **بعد از Build:** برای اجرای کانتینر از دستور Run استفاده می‌کنیم.

---

## 🔹 مرحله ۳ — تست لوکال Docker Container

بعد از Build اجرا کن:

```bash
docker run -d -p 8080:80 --name azuredemoapi azuredemoapi:latest
```

* Container روی پورت 8080 سیستم تو اجرا می‌شود
* حالا مرورگر یا Postman را باز کن به:

```
http://localhost:8080/api/hello
```

* باید خروجی `"Hello from AzureDemoApi!"` را ببینی ✅

---

🎉 تا اینجا پروژه **Dockerized** شده و لوکال تست شد.

---

## 🔹 مرحله ۴ — قدم بعدی

قدم بعدی:

- **Push Docker Image به Azure Container Registry**
- **ساخت Pipeline CI/CD برای اتوماتیک کردن Build و Deploy**

💡 **نکته:** چون همه چیز Dockerized شده، می‌توانیم ابتدا همه را لوکال تست کنیم و بعد به Azure بفرستیم.

---

✅ این فایل README کامل، مرحله به مرحله و آماده انتشار روی GitHub است.

