
# آشنایی با `SupervisorJob` و `supervisorScope` در Kotlin Coroutines

در برنامه‌نویسی همزمان با Kotlin Coroutines، مدیریت وظایف به صورت مؤثر و قابل‌اعتماد اهمیت زیادی دارد. ابزارهای `SupervisorJob` و `supervisorScope` در این زمینه نقش کلیدی ایفا می‌کنند و به شما امکان می‌دهند وظایف همزمان را به شکلی مدیریت کنید که خطاهای یک وظیفه تأثیری بر دیگر وظایف نداشته باشند. در این مقاله به بررسی دقیق این دو ابزار، کاربردها، تفاوت‌ها، مثال‌های عملی، اشتباهات رایج و بهترین روش‌ها می‌پردازیم.

---

## `SupervisorJob` چیست؟

### تعریف و هدف
`SupervisorJob`
یک پیاده‌سازی خاص از رابط `Job` در Kotlin Coroutines است. این ابزار به شما امکان می‌دهد که وظایف فرزند (child coroutines) به صورت مستقل اجرا شوند. به این معنا که اگر یکی از وظایف دچار خطا شود یا لغو شود، تأثیری روی وظایف دیگر یا وظیفه اصلی ندارد.

### چرا از `SupervisorJob` استفاده کنیم؟
هدف اصلی `SupervisorJob` مدیریت خطاها در عملیات همزمان است. با این ابزار، می‌توانید سیاست‌های خطای اختصاصی برای هر وظیفه فرزند تعریف کنید و مطمئن شوید که شکست یک وظیفه به اجرای بقیه وظایف لطمه نمی‌زند.

---

## چگونه کار می‌کند؟

### مدیریت وظایف و کنترل خطاها
- **خطاهای مستقل:** در `SupervisorJob`، هر وظیفه فرزند می‌تواند بدون تأثیر روی وظایف دیگر شکست بخورد.
- **عدم گسترش خطا:** خطای یک وظیفه فرزند به وظیفه والد منتقل نمی‌شود، مگر اینکه وظیفه والد خودش دچار خطا یا لغو شود.
- **مدیریت خطاها:** برای مدیریت خطاها می‌توانید از `try-catch` در داخل وظایف فرزند استفاده کنید یا یک `CoroutineExceptionHandler` به زمینه (context) اضافه کنید.

**مثال:**
```kotlin
val supervisorJob = SupervisorJob()
val scope = CoroutineScope(Dispatchers.IO + supervisorJob)

scope.launch {
    println("وظیفه ۱ در حال اجرا است")
    delay(1000)
    println("وظیفه ۱ تکمیل شد")
}

scope.launch {
    println("وظیفه ۲ در حال اجرا است")
    throw Exception("وظیفه ۲ شکست خورد")
}
```
در این مثال، وظیفه ۱ به اجرای خود ادامه می‌دهد حتی اگر وظیفه ۲ شکست بخورد.

**منبع:** [Tech Your Chance](https://www.techyourchance.com/kotlin-coroutines-supervisorjob/)

---

## `supervisorScope` چیست؟

### تعریف و هدف
`supervisorScope`
یک سازنده سطح بالای کروروتین است که به طور خودکار یک `SupervisorJob` را در زمینه خود ایجاد می‌کند. در این محدوده، وظایف فرزند مانند `SupervisorJob` به صورت مستقل اجرا می‌شوند.

### چرا از `supervisorScope` استفاده کنیم؟
این ابزار زمانی مفید است که نیاز دارید چندین وظیفه همزمان را به شکلی ساختاری مدیریت کنید و نیازی به ایجاد دستی `SupervisorJob` ندارید.

**مثال:**
```kotlin
supervisorScope {
    launch {
        println("وظیفه ۱ در حال اجرا است")
        delay(1000)
        println("وظیفه ۱ تکمیل شد")
    }

    launch {
        println("وظیفه ۲ در حال اجرا است")
        throw Exception("وظیفه ۲ شکست خورد")
    }
}
```
در اینجا نیز شکست وظیفه ۲ تأثیری بر اجرای وظیفه ۱ ندارد.

**منبع:** [Kotlin Official Documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.supervisor-scope.html)

---

## تفاوت بین `SupervisorJob` و `supervisorScope`

| ویژگی                     | `SupervisorJob`                                    | `supervisorScope`                                |
|-----------------------------|---------------------------------------------------|--------------------------------------------------|
| **نوع**                   | یک پیاده‌سازی از `Job`                            | یک سازنده کروروتین                                |
| **استفاده**               | برای ایجاد دستی یک `CoroutineScope`               | برای مدیریت همزمانی به صورت ساختاری              |
| **رفتار**                 | وظایف فرزند به صورت مستقل اجرا می‌شوند             | وظایف فرزند به صورت مستقل اجرا می‌شوند            |
| **تنظیمات**               | نیاز به تنظیم دستی `CoroutineScope`               | به طور خودکار شامل یک `SupervisorJob` است         |
| **سهولت استفاده**         | نیاز به تنظیمات دستی                              | ساده‌تر و خواناتر                                |

**منبع:** [Outcome School](https://outcomeschool.com/blog/coroutinescope-vs-supervisorscope)

---

## کاربردهای عملی

### سناریوهای واقعی
- **درخواست‌های شبکه مستقل:** در برنامه‌هایی که چندین درخواست شبکه به صورت همزمان ارسال می‌شوند، شکست یکی نباید دیگران را متوقف کند.
- **عملیات رابط کاربری:** در توسعه اندروید، شکست در بارگذاری یک تصویر نباید مانع از نمایش دیگر داده‌ها شود.

**مثال کد عملی:**
```kotlin
val supervisor = SupervisorJob()
val scope = CoroutineScope(Dispatchers.IO + supervisor)

scope.launch {
    println("وظیفه ۱ در حال اجرا است")
    delay(1000)
    println("وظیفه ۱ تکمیل شد")
}

scope.launch {
    println("وظیفه ۲ در حال اجرا است")
    throw Exception("وظیفه ۲ شکست خورد")
}
```
**توضیح:**
- وظیفه ۱ به اجرای خود ادامه می‌دهد حتی اگر وظیفه ۲ شکست بخورد.

**منبع:** [Tech Your Chance](https://www.techyourchance.com/kotlin-coroutines-supervisorjob/)

---

## اشتباهات رایج

1. **مدیریت نکردن استثناها:** عدم استفاده از `try-catch` یا `CoroutineExceptionHandler` می‌تواند منجر به خطاهای غیرمنتظره شود.
2. **استفاده نادرست از `Job` و `SupervisorJob`:** استفاده مستقیم از این ابزارها بدون تنظیمات مناسب ممکن است تأثیرات مورد نظر را نداشته باشد.
3. **استفاده از `GlobalScope`:** این کار می‌تواند باعث نشت حافظه شود.

**منبع:** [Lukas Lechner](https://www.lukaslechner.com/7-common-mistakes-you-might-be-making-when-using-kotlin-coroutines/)

---

## بهترین روش‌ها

1. **استفاده از `SupervisorJob` برای وظایف مستقل:** وقتی نیاز دارید وظایف مستقل از یکدیگر باشند، از `SupervisorJob` استفاده کنید.
2. **پیاده‌سازی همزمانی ساختاری:** محدوده‌هایی تعریف کنید که به چرخه حیات اجزا متصل باشند.
3. **مدیریت دقیق استثناها:** از بلوک‌های `try-catch` یا `CoroutineExceptionHandler` استفاده کنید.

**منبع:** [Kotlin Official Documentation](https://kotlinlang.org/docs/exception-handling.html)

---

با درک صحیح `SupervisorJob` و `supervisorScope`، می‌توانید برنامه‌هایی مقاوم و قابل‌اعتماد طراحی کنید که در برابر خطاها عملکرد مطلوبی داشته باشند.



من برداشتم این هست که 

supervisorScope

در واقع یک اسکوپ معمولی هست با این تغاوت که بجای اینکه مجبور بشیم خودمون دستی 

SupervisorJob

ایجاد کنیم خودش پیشفرض  این رو برامون ایجا میکنه و بجای job

SupervisorJob
بر میگردونه



بله، برداشت شما کاملاً درست است!

  

supervisorScope
در حقیقت یک **اسکوپ استاندارد** در Kotlin Coroutines است، اما با این تفاوت که به صورت پیش‌فرض **یک** 
SupervisorJob 
برای شما ایجاد می‌کند. یعنی نیازی نیست که خودتان یک SupervisorJob دستی بسازید و به اسکوپ اضافه کنید. در واقع:

• وقتی از supervisorScope استفاده می‌کنید:

1. خودش به‌صورت خودکار یک SupervisorJob درون خود ایجاد می‌کند.

2. وظایف فرزندی که در این اسکوپ تعریف می‌کنید، به دلیل وجود SupervisorJob به‌صورت **مستقل** اجرا می‌شوند.

3. اگر یک وظیفه دچار خطا شود، تأثیری روی وظایف دیگر یا اسکوپ والد نمی‌گذارد.

  

به زبان ساده، supervisorScope **کار را راحت‌تر می‌کند** چون دیگر لازم نیست:

• دستی یک CoroutineScope بسازید.

• یک SupervisorJob به آن اضافه کنید.

  

**مثال:**

  

اگر بخواهید همان رفتار supervisorScope را با CoroutineScope و SupervisorJob پیاده کنید، کدتان به این شکل خواهد بود:

  
```

val supervisorJob = SupervisorJob()

val scope = CoroutineScope(Dispatchers.Default + supervisorJob)

  

scope.launch {

    _// Task 1_

}

  

scope.launch {

    _// Task 2_

}

```
  

اما با استفاده از supervisorScope، این کد ساده‌تر می‌شود:

  

```
supervisorScope {

    launch {

        _// Task 1_

    }

  

    launch {

        _// Task 2_

    }

}

```
  

**نکته:**

• در supervisorScope نیازی به مدیریت جداگانه Job یا Scope ندارید، و فقط یک بلوک ساده برای مدیریت وظایف همزمان ایجاد می‌کنید.

• اما اگر بخواهید یک اسکوپ مستقل و سفارشی ایجاد کنید، ممکن است همچنان نیاز به استفاده از SupervisorJob به‌صورت مستقیم داشته باشید.

  

این سادگی استفاده یکی از مزایای اصلی supervisorScope است! 😊