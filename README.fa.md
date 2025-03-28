# Django Iran SMS

## معرفی

سیستم یکپارچه‌سازی پیامک مبتنی بر جنگو برای ساده‌سازی استفاده از پیامک‌های داخلی ایران، با استفاده از سرویس‌های `parsianwebco.ir` و `melipayamak.com` و احراز هویت با JWT. این پکیج توسط تیم چلسرو توسعه داده شده و برای نسخه‌های آینده از سرویس‌های اضافی پشتیبانی خواهد کرد.

## ویژگی‌ها

- یکپارچه‌سازی با `parsianwebco.ir` و `melipayamak.com`
- احراز هویت مبتنی بر JWT با استفاده از `rest_framework_simplejwt`
- مقیاس‌پذیری و قابلیت توسعه برای سایر ارائه‌دهندگان پیامک
- نصب و پیکربندی آسان
## نصب

### پیش‌نیازها

- پایتون ۳.۱۱
- جنگو ۵.۱ یا بالاتر

### نصب از طریق pip

```bash
pip install django-iran-sms
```

### پیکربندی
در فایل settings.py پروژه جنگو خود، پارامترهای زیر را اضافه کنید:

### settings.py

```bash
INSTALLED_APPS = [
...
'drfiransms', # هنگام استفاده در DRF.

]
```

```bash
DJANGO_IRAN_SMS = {
    'AUTHENTICATION': 'rest_framework_simplejwt',  # مشخص کردن روش احراز هویت
    'SMS_BACKEND': 'PARSIAN_WEBCO_IR',  # تعیین ارائه‌دهنده پیامک
    'OTP_CODE': {
        'LENGTH': 8,  # طول پیش‌فرض کد OTP
        'EXPIRE_PER_MINUTES': 4,  # زمان انقضای پیش‌فرض کد در دقیقه
    },
    'PARSIAN_WEBCO_IR': {
        'API_KEY': 'API_KEY obtained from sms.parsianwebco.ir',  دریافت می‌شود # کلید API از ارائه‌دهنده پیامک
        'TEMPLATES': {
            'OTP_CODE': 1,  # شناسه قالب برای کد OTP
        }
    },
    'MELI_PAYAMAK_COM': {
        'USERNAME': 'Username used to log in to the melipayamak.com website.',
        'PASSWORD': 'API_KEY obtained from melipayamak.com',
        'FROM': '50004001001516',  # شماره فرستنده که باید از سرویس وب دریافت شود.
    }
}
```

## استفاده
### پیکربندی URL
در فایل urls.py خود، نمای‌های زیر را اضافه کنید:

- OTPCodeSend: برای فرستادن کد OTP.
- Authentication: برای مدیریت احراز هویت و ثبت‌نام اختیاری.
- MessageSend: برای فرستادن پیامک

### urls.py
```bash
from drfiransms.views import OTPCodeSend, Authentication, MessageSend

urlpatterns += [
    path('lur/send-code/', OTPCodeSend.as_view(), name='send_code'),  # وب سرویس فرستادن OTP Code
    path('lur/authentication/', Authentication.as_view(), name='authentication'),  # وب سرویس احراز هویت ، ورود و ثبت نام بصورت اتوماسیون
    path('lur/send-message/', MessageSend.as_view(), name='send_message'), # وب سرویس فرستادن پیامک
]
```

## فرستادن کد تایید از طریق API
برای فرستادن درخواست POST به منظور دریافت کد تایید برای یک شماره موبایل، از ساختار زیر استفاده کنید:


```bash
curl -X POST https://djangoiransms.chelseru.com/lur/send-code/ \
     -H "Content-Type: application/json" \
     -d '{
           "mobile": "09123456789"
         }'
```
```bash
curl -X POST https://djangoiransms.chelseru.com/lur/authentication/ \
     -H "Content-Type: application/json" \
     -d '{
           "mobile": "09123456789",
           "code": "108117114",
           "group": "1"
         }'
```

```bash
curl -X POST https://djangoiransms.chelseru.com/lur/send-message/ \
     -H "Content-Type: application/json" \
     -d '{
           "mobile_number": "09123456789",
           "message_text": "hello luristan."
         }'
```
## جدول کاربران
پکیج DjangoSMS به‌طور خودکار یک جدول کاربران در پنل مدیریت جنگو ایجاد می‌کند با دو فیلد:

- mobile: شماره موبایل کاربر را ذخیره می‌کند.
- user: یک رابطه یک‌به‌یک با مدل پیش‌فرض کاربر در جنگو.
- group: این ویژگی برای مواقعی است که نیاز به تقسیم‌بندی کاربران وجود دارد، به عنوان مثال یک کاربر با شماره موبایل می‌تواند هم به عنوان راننده و هم به عنوان مسافر ثبت‌نام کند. این ویژگی اختیاری است و اگر وارد نشود، مقدار پیش‌فرض ۰ خواهد بود.

## جدول کد OTP
پکیج drfsms به‌طور خودکار یک جدول کد OTP در پنل مدیریت جنگو ایجاد می‌کند با دو فیلد:
- mobile: شماره موبایل کاربر را ذخیره می‌کند.
- code: کد OTP را ذخیره می‌کند.
  
## احراز هویت JWT
پکیج Djangoiransms از احراز هویت JWT با استفاده از پکیج rest_framework_simplejwt پشتیبانی می‌کند. سیستم از این روش احراز هویت برای ارتباط امن با دروازه پیامک استفاده می‌کند. روش‌های دیگر احراز هویت و ورود به سیستم در حال توسعه هستند.

## برنامه‌های آینده
- پشتیبانی از ارائه‌دهندگان پیامک اضافی.
- بهبود مدیریت خطا.
- محدودیت نرخ و نظارت.
- مشارکت


این یک پکیج جنگو برای یکپارچه‌سازی بدون دردسر با سرویس‌های پیامک ایرانی مانند ParsianWebCo و Melipayamak است. مشارکت‌ها خوش‌آمد است! لطفاً درخواست‌های کشش یا مشکلات را در مخزن گیت‌هاب ارسال کنید.

## مجوز
MIT License
