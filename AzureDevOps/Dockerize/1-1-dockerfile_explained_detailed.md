# توضیح کامل Dockerfile پروژه AzureDemoApi

بیایم **هر خط از Dockerfile** را مرحله‌به‌مرحله و با جزئیات توضیح بدهیم تا ساختار و دلیل هر دستور کامل مشخص باشد، همراه با سینتکس هر دستور:

---

## Stage 1: Build

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
```
* **سینتکس:** `FROM <image>[:<tag>] AS <name>`
* این خط میگه که **یک Image پایه بسازیم برای Build**.
* `mcr.microsoft.com/dotnet/sdk:9.0` شامل **.NET SDK 9** است، یعنی تمام ابزارهای توسعه مثل `dotnet build`, `dotnet restore`, `dotnet publish` داخلش موجود هستند.
* `AS build` نام این مرحله را مشخص می‌کند تا بعداً بتوانیم فایل‌ها را از این مرحله به مرحله‌ی بعدی کپی کنیم (**Multi-stage build**).

```dockerfile
WORKDIR /app
```
* **سینتکس:** `WORKDIR <path>`
* مسیر کاری داخل کانتینر را تعیین می‌کند.
* دستورات بعدی (`COPY`, `RUN`) نسبت به این مسیر اجرا می‌شوند.
* اگر `/app` وجود نداشته باشد، Docker آن را می‌سازد.

```dockerfile
COPY *.csproj ./
```
* **سینتکس:** `COPY <src> <dest>`
* فقط فایل `.csproj` پروژه را به کانتینر کپی می‌کند.
* دلیل کپی جداگانه: **استفاده بهینه از Docker cache**.
  * اگر فقط کد تغییر کند اما پکیج‌ها (`NuGet packages`) تغییر نکرده باشند، مرحله `dotnet restore` از کش استفاده می‌کند و سریع‌تر Build می‌شود.

```dockerfile
RUN dotnet restore
```
* **سینتکس:** `RUN <command>`
* تمام **پکیج‌های NuGet** مورد نیاز پروژه را دانلود و Cache می‌کند.
* این مرحله تضمین می‌کند که تمام وابستگی‌ها قبل از Build آماده هستند.

```dockerfile
COPY . ./
```
* **سینتکس:** `COPY <src> <dest>`
* حالا کل کد پروژه (Controllers، Models، فایل‌ها و فولدرها) را به کانتینر کپی می‌کند.
* این مرحله بعد از `restore` است تا Docker layer cache بهینه شود.

```dockerfile
RUN dotnet publish -c Release -o out
```
* **سینتکس:** `RUN <command>`
* پروژه را در حالت **Release** Build می‌کند.
* خروجی Publish در مسیر `/app/out` قرار می‌گیرد.
* خروجی شامل تمام DLL ها، فایل‌های پیکربندی و منابع لازم برای اجرا است.

---

## Stage 2: Runtime

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:9.0
```
* **سینتکس:** `FROM <image>[:<tag>]`
* این مرحله یک Image سبک برای اجرای پروژه ایجاد می‌کند.
* شامل فقط **Runtime** .NET 9 است، **نه SDK**.
* از آنجا که پروژه قبلاً Build و Publish شده، نیازی به ابزار Build در این Image نیست → Image سبک‌تر می‌شود.

```dockerfile
WORKDIR /app
```
* **سینتکس:** `WORKDIR <path>`
* مسیر کاری Runtime.
* فایل‌های Publish شده از مرحله قبل به این مسیر کپی می‌شوند.

```dockerfile
COPY --from=build /app/out ./
```
* **سینتکس:** `COPY [--from=<name>] <src> <dest>`
* فایل‌های منتشر شده (`Publish`) از **مرحله Build** را به این Image Runtime کپی می‌کند.
* `--from=build` مشخص می‌کند که فایل‌ها را از مرحله‌ای که `AS build` نام داشت برداریم.
* این روش **Multi-stage build** باعث می‌شود فقط فایل‌های لازم در Image نهایی باشند.

```dockerfile
EXPOSE 80
```
* **سینتکس:** `EXPOSE <port>`
* پورت HTTP که کانتینر روی آن Listen می‌کند.
* وقتی Run می‌کنی، می‌توانی این پورت را به پورت دلخواه سیستم Host وصل کنی، مثلاً:
  ```bash
  docker run -p 8080:80 myapi:latest
  ```

```dockerfile
ENTRYPOINT ["dotnet", "AzureDemoApi.dll"]
```
* **سینتکس:** `ENTRYPOINT ["executable", "param1", ...]` یا `ENTRYPOINT <command>`
* دستور پیش‌فرض اجرای کانتینر.
* وقتی کانتینر اجرا می‌شود، این خط پروژه Publish شده را اجرا می‌کند.
* `ENTRYPOINT` باعث می‌شود کانتینر دقیقا مثل اجرای `dotnet AzureDemoApi.dll` در ترمینال عمل کند.

---

### جمع‌بندی

* **Stage 1: Build**
  * Build و Publish پروژه با SDK
  * Cache بهینه با کپی جداگانه `.csproj`
  * خروجی Publish آماده برای اجرای نهایی

* **Stage 2: Runtime**
  * Image سبک فقط با Runtime
  * فایل‌های Publish شده از مرحله Build کپی می‌شوند
  * پورت 80 باز و ENTRYPOINT مشخص است

---

💡 نکته: می‌توان برای درک بهتر یک **Diagram تصویری ساده از این دو Stage و جریان فایل‌ها** کشید تا ببینی چه اتفاقی داخل هر مرحله می‌افتد.

