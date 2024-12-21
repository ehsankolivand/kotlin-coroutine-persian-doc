**delay در Kotlin Coroutines: مقدمه‌ای کامل**

  

**مقدمه**

  

در برنامه‌نویسی همزمان (Concurrent Programming)، یکی از چالش‌ها مدیریت زمان توقف یا تأخیر بدون انسداد (Blocking) است. Kotlin Coroutines با معرفی تابع delay این امکان را فراهم می‌کند که برنامه‌ها بدون مسدود کردن منابع سیستم، اجرای وظایف را به تعویق بیاندازند.

  

این مقاله با رویکردی جامع به بررسی مفهوم delay، مزایا، مثال‌های عملی، بهترین روش‌ها و اشتباهات رایج می‌پردازد.

  

**مفهوم delay و تعلیق غیرمسدودکننده**

  

delay


یک تابع **تعلیق‌کننده (Suspending Function)** است. این تابع به جای توقف کل نخ (Thread)، تنها اجرای Coroutine را متوقف می‌کند. در طول این توقف، Thread آزاد می‌شود تا سایر وظایف یا Coroutine‌ها اجرا شوند.

  

**مراحل عملکرد delay**

1. **تعلیق Coroutine**: Coroutine جاری متوقف می‌شود.

2. **آزادسازی Thread**: Thread به سایر وظایف تخصیص می‌یابد.

3. **از سرگیری Coroutine**: پس از پایان زمان تأخیر، Coroutine به وضعیت قبلی خود بازمی‌گردد.

  

این رفتار باعث افزایش کارایی و پاسخ‌گویی برنامه‌ها می‌شود.

  

**مثال‌های کاربردی**

  

**مثال ۱: استفاده ساده از delay**

  

```
import kotlinx.coroutines.*

  

fun main() = runBlocking {

    println("شروع کار: ${Thread.currentThread().name}")

    delay(2000L) _// تأخیر ۲ ثانیه بدون انسداد_

    println("ادامه کار: ${Thread.currentThread().name}")

}

```
  

در این مثال، delay اجرای Coroutine را به مدت ۲ ثانیه متوقف می‌کند، اما نخ مسدود نمی‌شود.

  

**مثال ۲: اجرای وظایف به ترتیب با delay**

  

```
import kotlinx.coroutines.*

  

fun main() = runBlocking {

    println("شروع وظیفه اول")

    delay(1000L) _// توقف ۱ ثانیه‌ای_

    println("پایان وظیفه اول")

  

    println("شروع وظیفه دوم")

    delay(2000L) _// توقف ۲ ثانیه‌ای_

    println("پایان وظیفه دوم")

}
```

  

در این مثال، وظایف به ترتیب اجرا شده و زمان‌های تأخیر بین آن‌ها مدیریت می‌شود.

  

**مثال ۳: اجرای همزمان چند Coroutine**

  

```
import kotlinx.coroutines.*

  

fun main() = runBlocking {

    val job1 = launch {

        delay(1000L)

        println("وظیفه ۱ تمام شد")

    }

  

    val job2 = launch {

        delay(2000L)

        println("وظیفه ۲ تمام شد")

    }

  

    joinAll(job1, job2) _// انتظار برای پایان هر دو وظیفه_

}

```
  

اینجا دو Coroutine به طور همزمان اجرا می‌شوند و نخ اصلی در طول زمان‌های تأخیر آزاد است.

  

**بهترین روش‌ها**

1. **جایگزینی Thread.sleep() با delay**: استفاده از delay در Coroutine‌ها به جای Thread.sleep() باعث جلوگیری از انسداد نخ‌ها می‌شود.

2. **مدیریت Cancellation**: از آنجا که delay قابل لغو است، حتماً کدهای مربوط به مدیریت لغو را اضافه کنید.

3. **استفاده از کوتاه‌ترین زمان ممکن برای تأخیر**: تأخیرهای طولانی می‌توانند کارایی برنامه را کاهش دهند.

  

**اشتباهات رایج**

1. **استفاده از Thread.sleep()**: این روش نخ را مسدود کرده و مزایای Coroutine‌ها را از بین می‌برد.

2. **عدم توجه به Context**: استفاده از Dispatcher نادرست ممکن است باعث کاهش کارایی یا مسدود شدن UI شود.

3. **بی‌توجهی به Cancellation**: اگر delay در Coroutine لغو شود و مدیریت مناسبی وجود نداشته باشد، رفتارهای غیرمنتظره رخ خواهد داد.

  

**مقایسه با روش‌های مشابه در زبان‌های دیگر**

  • **Kotlin (delay):**

• مسدودکننده: خیر

• محدوده عملکرد: Coroutine Scope

• بازگشت به اجرا: Coroutine Scheduler

• **Java (Thread.sleep):**

• مسدودکننده: بله

• محدوده عملکرد: Thread Scope

• بازگشت به اجرا: Thread Scheduler


**نتیجه‌گیری**
  
تابع delay ابزاری قدرتمند برای ایجاد برنامه‌های کارآمد و واکنش‌گرا است. با استفاده صحیح از این تابع و رعایت بهترین روش‌ها، می‌توان از تمام قابلیت‌های Kotlin Coroutines بهره‌مند شد.


**منابع**

1. [Kotlin Documentation](https://kotlinlang.org)

2. [Droidcon Article on Coroutines](https://droidcon.com)

3. [Baeldung Kotlin Tutorials](https://www.baeldung.com/kotlin)