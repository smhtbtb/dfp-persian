<h1 dir="rtl">
فصل 12
</h1>

<h2 dir="rtl">
آپلود تصویر و فایل
</h2>

<p dir="rtl">
ما قبلتر در فصل 6 منابع static را، مانند عکس پیکربندی کردیم؛ اما فایلهای بارگذاری شده توسط کاربر، همچون جلد کتاب، کمی متفاوت است. برای شروع، جنگو به قالب static اشاره میکند، در حالی که هر چیزی که توسط user آپلود میشود، چه فایل و چه عکس، media نامیده میشود.
</p>
<p dir="rtl">
روند افزودن این ویژگیها برای فایلها یا تصاویر مشابه هست؛ اما، برای تصاویر، باید کتابخانه پردازش تصویر پایتون به نام Pillow را نصب کرد که شامل ویژگیهای اضافه مانند اعتبارسنجی اولیه است.
</p>
<p dir="rtl">
بیایید Pillow را با استفاده از روش آشنای خود در Docker نصب کنیم، container هایمان را متوقف کنیم، و ساخت image جدیدی را force کنیم.
</p>
<h4 dir="rtl">
خط فرمان (Command line)
</h4>

```
$ docker-compose exec web pipenv install pillow==7.2.0
$ docker-compose down
$ docker-compose up -d --build
```

<br>

<h3 dir="rtl">
فایلهای Media
</h3>
<p dir="rtl">
فایل های static قابل اطمینان هستند در صورتی که فایل های مدیا غیرقابل اطمینان میباشد. همیشه نگرانی های امنیتی زمان مواجهه با محتوای بارگذاری شده توسط user وجود دارد. قابل ذکر است که اعتبارسنجی همه‌ی فایلهای آپلود شده برای اطمینان حاصل کردن از اینکه آنها همان چیزی هستند که میگویند، مهم است. چندین روش وجود دارد که یک فرد مخرب میتواند به وبسایتی که کورکورانه آپلودهای کاربران را میپذیرد ،حمله کند.
</p>
<p dir="rtl">
برای شروع بیایید دو کانفیگ جدید به فایل config/settings.py اضافه کنیم. به طور پیش فرض MEDIA_URL  و MEDIA_ROOT داخل فایل settings وجود ندارند، پس ما نیاز به کانفیگ کردن آنها داریم.
</p>
<p dir="rtl">
MEDIA_ROOT  مسیر سیستمی دقیق به پوشه های فایلهای بارگذاری شده توسط کاربر است.
</p>
<p dir="rtl">
MEDIA_URL  نشانی اینترنتی (url) است که میتوانیم در templat های خود برای فایلهایمان استفاده کنیم.
</p>
<p dir="rtl">
ما میتوانیم هر دوی این تنظیمات را بعد از STATICFILES_FINDERS در انتهای فایل config/settings.py اضافه کنیم. برای راحتی کار هردو را media نامگذاری میکنیم. فراموش نکنید اسلش (/) را برای MEDIA_URL قرار دهید.
</p>
<h4 dir="rtl">
کد
</h4>

```python
# config/settings.py
MEDIA_URL = '/media/' # new
MEDIA_ROOT = str(BASE_DIR.joinpath('media')) # new
```

<p dir="rtl">
سپس یک پوشه جدید به نام media و یک پوشه دیگر به نام covers در داخل آن اضافه میکنیم.
</p>
<h4 dir="rtl">
خط فرمان (Command line)
</h4>

```
$ mkdir media
$ mkdir media/covers
```

<p dir="rtl">
و سرانجام، از آنجایی که فرض بر این است که محتوای بارگذاری شده توسط کاربر در یک production context  (محیط عملی) وجود دارد، برای مشاهده آیتم های local، نیاز داریم تا فایل config/urls.py هم به روزرسانی کنیم تا فایلها به صورت local نمایش داده شوند. این مرحله شامل import کردن settings  و static در بالا و اضافه کردن یک خط کد در پایین فایل میشود.
</p>
<h4 dir="rtl">
کد
</h4>

```python
# config/urls.py
from django.conf import settings # new
from django.conf.urls.static import static # new
from django.contrib import admin
from django.urls import path, include
urlpatterns = [
  # Django admin
  path('admin/', admin.site.urls),

  # User management
  path('accounts/', include('allauth.urls')),

  # Local apps
  path('', include('pages.urls')),
  path('books/', include('books.urls')),
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT) # new
```

<br>

<h3 dir="rtl">
مدل‌ها
</h3>
<p dir="rtl">
با پیکربندی های عمومی media اکنون ما میتوانیم به مدلهای خود مراجعه کنیم. برای ذخیره سازی تصاویر از ImageField جنگو استفاده میکنیم که خود شامل برخی از اعتبارسنجی های اولیه پردازش تصویر میباشد.
نام فیلد cover (جلد) میباشد و ما پوشه‌ی MEDIA_ROOT/covers را برای بارگذاری تصاویر انتخاب میکنیم (قسمت MEDIA_ROOT بر اساس تنظیمات قبلی فایل settings.py انجام شده است).
</p>
<h4 dir="rtl">
کد
</h4>

```python
# books/models.py
class Book(models.Model):
  id = models.UUIDField(
    primary_key=True,
    default=uuid.uuid4,
    editable=False)
  title = models.CharField(max_length=200)
  author = models.CharField(max_length=200)
  price = models.DecimalField(max_digits=6, decimal_places=2)
  cover = models.ImageField(upload_to='covers/') # new

  def __str__(self):
    return self.title

  def get_absolute_url(self):
    return reverse('book_detail', kwargs={'pk': str(self.pk)})
```
<center>
<p dir="rtl">
اگر میخواستیم آپلود یک فایل معمولی را به جای یک فایل تصویری مجاز کنیم، تنها تفاوت تغییر ImageField به FileField بود.
</p>
</center>

<p dir="rtl">
تا اینجا مدل را آپدیت کردیم. وقت این است که یک فایل migrations بسازیم.
</p>
<h4 dir="rtl">
خط فرمان (Command line)
</h4>

```
$ docker-compose exec web python manage.py makemigrations books
You are trying to add a non-nullable field 'cover_image' to book without a default; we can't do that (the database needs something to populate existing rows).
Please select a fix:
1) Provide a one-off default now (will be set on all existing rows with a null value for this column)
2) Quit, and let me add a default in models.py
Select an option:
```
<p dir="rtl">
اوپس! چی شد؟ ما داریم یک فیلد جدید به دیتابیس اضافه میکنیم، اما قبلا برای هر کتاب سه ورودی در دیتابیس داشتیم. هنوز نتوانستیم مقدار پیش فرض را برای جلد تنظیم کنیم.
</p>
<p dir="rtl">
برای حل این مشکل، گزینه 2 را انتخاب میکنیم و یک فیلد blank به فیلد تصویرمان اضافه میکنیم و آنرا برابر True قرار میدهیم.
</p>
<h4 dir="rtl">
کد
</h4>

```python
# books/models.py
cover = models.ImageField(upload_to='covers/', blank=True) # new
```
<center>
<p dir="rtl">
این اتفاق رایج است که blank و null با هم برای تعیین مقدار پیش فرض در یک فیلد استفاده می شوند. یک گاتچا این است که نوع فیلد - ImageField در مقابل CharField و غیره - نحوه استفاده صحیح از آنها را نشان میدهد، بنابراین داکیومنتهای این بخش را برای استفاده در آینده نزدیک مطالعه کنید.
</p>
</center>

<p dir="rtl">
حال میتوانیم یک فایل migrations بدون ارور بسازیم.
</p>
<h4 dir="rtl">
خط فرمان (Command line)
</h4>

```
$ docker-compose exec web python manage.py makemigrations books
```
<p dir="rtl">
و سپس migration (مهاجرت) را به پایگاه داده ما را اعمال کنید.
</p>
<h4 dir="rtl">
خط فرمان (Command line)
</h4>

```
$ docker-compose exec web python manage.py migrate
```

<br>

<h3 dir="rtl">
ادمین
</h3>

