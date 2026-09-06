# اتصال دو دامنه

دامنه اصلی سایت: `https://tavonikohenor.ir/`
دامنه دوم برای انتقال خودکار: `https://taavonikohnoor.ir/`

## 1) GitHub Pages

فایل `CNAME` فقط باید این مقدار را داشته باشد:

```text
tavonikohenor.ir
```

در Settings → Pages → Custom domain همین دامنه را ثبت کن و **Enforce HTTPS** را فعال کن. GitHub Pages برای یک سایت فقط یک CNAME اصلی نگه می‌دارد؛ دامنه دوم باید با Redirect در DNS/Cloudflare به دامنه اصلی منتقل شود.

## 2) DNS دامنه اصلی

برای `tavonikohenor.ir`:

- `A` / `@` → `185.199.108.153`
- `A` / `@` → `185.199.109.153`
- `A` / `@` → `185.199.110.153`
- `A` / `@` → `185.199.111.153`
- `CNAME` / `www` → `farrokhzad743-dot.github.io`

## 3) دامنه دوم در Cloudflare

برای `taavonikohnoor.ir` چون فقط alias است، آن را به سایت اصلی Redirect کن. ساده‌ترین روش Cloudflare Redirect Rules است.

DNS دامنه دوم را طوری بساز که Cloudflare بتواند درخواست را بگیرد؛ طبق مستندات Cloudflare برای دامنه alias می‌توان برای `@` و `www` رکورد A با مقدار `192.0.2.1` و وضعیت **Proxied** ساخت. این IP به origin واقعی متصل نمی‌شود و برای اعمال Redirect Rules استفاده می‌شود.

رکوردها:

- `A` / `@` → `192.0.2.1` — **Proxied**
- `A` / `www` → `192.0.2.1` — **Proxied**

سپس Rules → Redirect Rules → Single Redirect بساز:

**When incoming requests match:**

```text
(http.host eq "taavonikohnoor.ir") or (http.host eq "www.taavonikohnoor.ir")
```

**Action:** Dynamic Redirect

**Expression:**

```text
concat("https://tavonikohenor.ir", http.request.uri.path)
```

Status: **301**

گزینه **Preserve query string** را روشن کن.

نتیجه:

- `taavonikohnoor.ir` → `https://tavonikohenor.ir/`
- `www.taavonikohnoor.ir` → `https://tavonikohenor.ir/`
- مسیرها و query stringها نیز حفظ می‌شوند.

## 4) HTTPS و امنیت

بعد از سبز شدن گواهی در GitHub Pages، Enforce HTTPS را روشن کن. سایت از CSP، Referrer Policy، noindex برای پنل مدیریت و RLS سمت Supabase استفاده می‌کند. کلید `service_role` یا رمز دیتابیس نباید هیچ‌وقت داخل پروژه قرار بگیرد.

## 5) Supabase

فایل `supabase-security-final.sql` را در SQL Editor همان پروژه Supabase اجرا کن. این نسخه مجوز تغییر داده‌ها را فقط به ایمیل‌های موجود در جدول خصوصی `owner_access` می‌دهد و مجوز عمومی خواندن داده‌های عمومی سایت را حفظ می‌کند.
