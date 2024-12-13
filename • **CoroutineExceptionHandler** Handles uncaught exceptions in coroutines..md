


# مدیریت استثناها در CoroutineExceptionHandler در Kotlin Coroutines

  

## ۱. مفهوم CoroutineExceptionHandler

  

### CoroutineExceptionHandler
چیست؟

- ****CoroutineExceptionHandler**** 

- این ابزار مشابه ****Thread.uncaughtExceptionHandler**** در جاوا عمل کرده و مکانیزمی متمرکز برای مدیریت استثناهای غیرمنتظره فراهم می‌کند.

  

### اهمیت در مدیریت استثناها

- در Kotlin Coroutines، استثناها می‌توانند در سلسله مراتب کوروتین‌ها گسترش پیدا کنند و باعث لغو یا خرابی ناخواسته شوند.

- ****CoroutineExceptionHandler**** 

- امکان تعریف رفتارهای سفارشی برای این استثناها را فراهم می‌کند، مثل ثبت گزارش یا انجام عملیات تمیزکاری.

  

### نحوه مدیریت استثناهای غیرقابل‌گرفتن

- وقتی استثنایی در یک کوروتین رخ دهد و گرفته نشود، ****CoroutineExceptionHandler**** آن را رهگیری کرده و می‌توان از آن برای لاگ‌برداری یا نمایش پیام به کاربر استفاده کرد.


### ارتباط با، Job، و SupervisorJob


- Job:

- استثناهای کوروتین فرزند به والد خود گسترش پیدا می‌کنند. اگر کوروتین فرزند شکست بخورد، می‌تواند والد خود را لغو کند مگر اینکه از ****SupervisorJob**** استفاده شود.

- ****SupervisorJob****: 

- استفاده از SupervisorJob باعث می‌شود شکست یک کوروتین بر دیگر کوروتین‌ها تأثیری نداشته باشد و امکان مدیریت مستقل استثناها فراهم شود.

  

---

  

## ۲. سناریوهای استفاده از CoroutineExceptionHandler

  

- ****ثبت استثناهای غیرقابل‌گرفتن در Global Scopes****
  ( استثناهای غیرقابل گرفتن به خطا ها و با استثنا هایی گفته میشه که توسط برنامه نویس پیش بینی نشده اند و هندل نشده اند یعنی در try catch   گذاشته نشده اند )

  - اتصال CoroutineExceptionHandler به یک کوروتین گلوبال، امکان ثبت لاگ مرکزی را برای دیباگ و پایش آسان‌تر فراهم می‌کند.

  

- ****مکانیزم‌های جایگزین برای عملیات‌های حیاتی****

  - در عملیات‌های مهم، می‌توان از این هندلر برای فعال‌سازی مکانیزم‌های جایگزین یا اجرای مجدد (retry) در هنگام برخورد با استثناها استفاده کرد.

  

- ****مدیریت خطاها در لایه‌های UI****

  - در اپلیکیشن‌های UI، به خصوص در اندروید، استفاده از این هندلر امکان مدیریت خطاها به صورت کاربرپسند (مثل نمایش پیام‌های خطای مناسب) را فراهم می‌کند.

  

- ****رفتار در Concurrency ساختار یافته****

  - ****با launch****: استثناها به والد گسترش پیدا می‌کنند و در صورت عدم مدیریت، به CoroutineExceptionHandler ارسال می‌شوند.

  - ****با async****: استثناها داخل شیء ****Deferred**** قرار می‌گیرند و باید با استفاده از `await()` گرفته شوند.

  - ****با supervisorScope****: استثناها در یک کوروتین تأثیری بر سایر کوروتین‌ها ندارد و CoroutineExceptionHandler می‌تواند بدون لغو کل اسکوپ، استثناهای غیرمنتظره را مدیریت کند.

  

---

  

## ۳. مثال‌های عملی با کد

  

### اتصال CoroutineExceptionHandler به یک CoroutineScope

```
import kotlinx.coroutines.*

  

fun main() = runBlocking {

    val handler = CoroutineExceptionHandler { _, exception ->

        println("Caught $exception")

    }

  

    val scope = CoroutineScope(Job() + handler)

  

    scope.launch {

        throw RuntimeException("Test Exception")

    }

  

    delay(100) // زمان برای اجرای کوروتین

}

```
  

**خروجی مورد انتظار:**

  

```
Caught java.lang.RuntimeException: Test Exception

```
  

**مدیریت استثناهای خاص و ارائه بازخورد**

  
```

import kotlinx.coroutines.*

  

fun main() = runBlocking {

    val handler = CoroutineExceptionHandler { _, exception ->

        when (exception) {

            is IllegalArgumentException -> println("Invalid argument: ${exception.message}")

            is ArithmeticException -> println("Math error: ${exception.message}")

            else -> println("Caught $exception")

        }

    }

  

    val scope = CoroutineScope(Job() + handler)

  

    scope.launch {

        throw IllegalArgumentException("Invalid input")

    }

  

    delay(100)

}

```
  

**خروجی مورد انتظار:**

  

```
Invalid argument: Invalid input
```

  

**ترکیب CoroutineExceptionHandler با SupervisorJob**

  

```
import kotlinx.coroutines.*

  

fun main() = runBlocking {

    val handler = CoroutineExceptionHandler { _, exception ->

        println("Caught $exception")

    }

  

    val supervisor = SupervisorJob()

    val scope = CoroutineScope(supervisor + handler)

  

    scope.launch {

        throw RuntimeException("Child coroutine failed")

    }

  

    scope.launch {

        delay(100)

        println("This coroutine completes successfully")

    }

  

    delay(200) _// زمان برای اجرای کوروتین‌ها_

}

```
  

**خروجی مورد انتظار:**

  

```
Caught java.lang.RuntimeException: Child coroutine failed

This coroutine completes successfully
```

  

**۴. بهترین روش‌ها و اشتباهات رایج**

  

**بهترین روش‌ها**

• **استفاده از بلوک‌های try-catch در داخل کوروتین‌ها**: برای مدیریت و بازیابی سریع استثناها از بلوک‌های try-catch استفاده کنید.

• **استفاده از SupervisorJob برای استقلال کوروتین‌ها**: برای جلوگیری از تأثیر شکست یک کوروتین بر دیگران، از SupervisorJob استفاده کنید.

• **ثبت لاگ مرکزی**: از CoroutineExceptionHandler برای ثبت لاگ‌های متمرکز استفاده کنید.

  

**اشتباهات رایج**

• **اتصال نادرست CoroutineExceptionHandler**: اتصال هندلر به کوروتین فرزند مؤثر نیست؛ اطمینان حاصل کنید که به والد یا اسکوپ اصلی متصل باشد.

• **نادیده‌گرفتن استثناها در async**: استثناها در async داخل Deferred قرار می‌گیرند و بدون استفاده از await() مدیریت نمی‌شوند.

  

**منابع و مستندات**

• [مستندات رسمی Kotlin Coroutines](https://kotlinlang.org/docs/exception-handling.html)

• [وبلاگ توسعه‌دهندگان Kotlin](https://blog.jetbrains.com/)

• [آموزش‌های مرتبط](https://maxkim.eu/things-every-kotlin-developer-should-know-about-coroutines-part-4-exception-handling)