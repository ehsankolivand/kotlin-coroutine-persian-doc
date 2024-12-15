# درک و استفاده از `await` در Coroutineهای کاتلین

  

## مقدمه

  

Coroutine

ها در کاتلین ابزاری قدرتمند برای برنامه‌نویسی همزمان (Concurrent) و غیربلاک‌کننده ارائه می‌دهند. در میان قابلیت‌های متعدد این چارچوب، تابع `await` جایگاه ویژه‌ای دارد. این تابع به شما امکان می‌دهد تا از عملیات‌های غیرهمزمان نتیجه دریافت کنید، بدون آن‌که اجرای رشته (Thread) اصلی را مسدود کنید.

  

در این مقاله، ابتدا مفهوم `await` و تعامل آن با `Deferred` و `async` را توضیح می‌دهیم، سپس به کاربردهای عملی، نکات عملکردی، اشتباهات رایج، و بهترین شیوه‌ها اشاره خواهیم کرد. در پایان نیز مثال‌هایی از ترکیب `await` در معماری‌های مدرن نظیر MVI و Jetpack Compose را خواهید دید.

  

---

  

## `await
` چیست و چرا مهم است؟

  

در کاتلین، زمانی که از `async` برای راه‌اندازی یک Coroutine غیرهمزمان استفاده می‌کنید، شیئی از نوع `Deferred<T>` برمی‌گرداند. این شیء در واقع یک "وعده" (Promise) برای تامین نتیجه در آینده است. برای دریافت نتیجه‌ی این عملیات، از تابع معلق `await` استفاده می‌کنیم. فراخوانی `await` روی یک `Deferred` باعث می‌شود Coroutine فراخواننده تا زمان تکمیل نتیجه صبر (Suspend) کند، اما رشته‌ی اصلی را بلاک نمی‌کند.

  

****ویژگی‌های کلیدی `await`:****  

- ****غیربلاک‌کننده:**** `await` باعث توقف رشته اصلی نمی‌شود، بلکه فقط Coroutine را معلق می‌کند. این یعنی سایر Coroutineها می‌توانند در همین حین ادامه کار دهند.  

- ****قابلیت لغو (Cancellable):**** اگر Coroutine فراخواننده لغو شود، `await` نیز یک `CancellationException` صادر می‌کند.  

- ****انتقال خطا (Error Propagation):**** اگر در جریان اجرای `async` استثنایی رخ دهد، هنگام فراخوانی `await` همان استثنا را دریافت می‌کنید و می‌توانید آن را مدیریت کنید.

  

---

  

## ارتباط `await` با `Deferred` و `async`

  

تابع `async` یک Coroutine جدید راه‌اندازی می‌کند که به صورت غیرهمزمان اجرا شده و نتیجه‌ای از نوع `Deferred<T>` بازمی‌گرداند. برای گرفتن نتیجه از این Coroutine، کافی است تابع `await()` را روی `Deferred` فراخوانی کنید.

  

****مراحل کار:****

  

1. با استفاده از `async` یک عملیات غیرهمزمان را آغاز کنید:

   ```

   val deferredResult: Deferred<Int> = async {

       // یک عملیات محاسباتی سنگین

       computeValue()

   }
```
```
  

2. با فراخوانی await روی deferredResult نتیجه را دریافت کنید:

  
```

val result: Int = deferredResult.await()
```

  

در اینجا، تا زمانی که محاسبه کامل نشود، Coroutine فراخواننده در نقطه‌ی await معلق می‌ماند، اما مسدود نشده و سایر Coroutineها می‌توانند ادامه دهند.

  

**کاربردهای عملی** **await**

  

**۱. فراخوانی API و دریافت داده از شبکه**

  

زمانی که درخواست‌های شبکه‌ای دارید، می‌توانید با async آن‌ها را به صورت همزمان آغاز کنید، سپس با await نتایجشان را دریافت کنید.

  

```
val userData: Deferred<String> = async(Dispatchers.IO) {

    _// شبیه‌سازی تاخیر در شبکه_

    fetchUserData()

}

val data = userData.await()

println("داده دریافتی: $data")

```
  

**۲. اجرای عملیات‌های همزمان**

  

اگر چندین وظیفه مستقل دارید که می‌توانند به صورت موازی انجام شوند:

  

```
val firstTask = async { performFirstTask() }

val secondTask = async { performSecondTask() }

  

val combinedResult = firstTask.await() + secondTask.await()

println("نتیجه‌ی نهایی: $combinedResult")

```
  

اینگونه، با اجرای همزمان کارها، کارایی بالاتری بدست می‌آورید و بدون بلاک کردن رشته اصلی، تا رسیدن پاسخ صبر می‌کنید.

  

**مقایسه** **await** **با** **join** **و** **launch**

  

تابع launch یک Coroutine راه می‌اندازد که **نتیجه‌ای بازنمی‌گرداند**. در حالی که async نتیجه باز می‌گرداند (Deferred<T>). برای این‌که تفاوت را روشن‌تر کنیم:

• launch **و** join**:**

launch
تنها یک Coroutine را شروع می‌کند بدون آن‌که نتیجه‌ای برگرداند.

  

```

val job = launch {

    performTask()

}

job.join() _// صبر تا تکمیل کار_
```



در اینجا join() فقط صبر می‌کند تا کار تمام شود، اما خروجی‌ای وجود ندارد.

  

• async **و** await**:**

async
یک Deferred برمی‌گرداند که حاوی نتیجه آینده است و await() آن نتیجه را دریافت می‌کند.




```
val deferred = async {

    computeValue()

}

val result = deferred.await()

println("نتیجه: $result")

```
  


  

  

**از دید کارایی:**

• async+await

برای زمانی مناسب‌اند که به نتیجه نیاز دارید.

• launch+join
زمانی کارامدند که صرفاً می‌خواهید اطمینان یابید عملیات تمام شده است و نیازی به نتیجه ندارید.

  

**نکات عملکردی و مدیریت خطا**

  

**خطاهای رایج**

• **فراموش کردن فراخوانی** await**:** اگر async را صدا بزنید اما هیچ‌گاه await نکنید، نتیجه هرگز دریافت نخواهد شد.

• **ایجاد بلاک شدن ناخواسته:** await اگر در یک بلوک runBlocking بدون توجه به زمینه مناسب فراخوانی شود، می‌تواند به بلاک شدن رشته اصلی بینجامد. از CoroutineScope و کانتکست‌های مناسب (مثلاً Dispatchers.IO برای کارهای ورودی/خروجی) استفاده کنید.

• **عدم مدیریت استثناها:** هرگز فرض نکنید که عملیات غیرهمزمان بدون خطا پیش می‌رود. حتماً‌ با try-catch خطاها را مدیریت کنید.

  

**بهترین شیوه‌ها**

• **استفاده از** viewModelScope**:** در معماری‌های اندروید، استفاده از viewModelScope برای راه‌اندازی Coroutineها کمک می‌کند عملیات را در طول چرخه عمر ViewModel کنترل کرده و در صورت نیاز لغو کنید.

• **تست قطعه به قطعه:** عملیات async/await را با تست‌های واحد و یکپارچه بررسی کنید تا عملکرد و مدیریت استثناها تضمین شود.

  

**نکات پیشرفته: به‌کارگیری** **await** **در معماری MVI و Jetpack Compose**

  

در معماری MVI (Model-View-Intent) و Jetpack Compose، از Coroutineها برای بارگذاری داده‌ها و مدیریت حالت (State) استفاده می‌کنیم:

• **مدیریت حالت در MVI:**

فرض کنید یک ViewModel دارید که داده را از شبکه دریافت کرده و در UiState قرار می‌دهد.

  

```
viewModelScope.launch {

    try {

        val result = async { fetchData() }.await()

        _uiState.value = UiState.Success(result)

    } catch (e: Exception) {

        _uiState.value = UiState.Error(e)

    }

}
```

  

اینجا await تضمین می‌کند پس از تکمیل کار غیرهمزمان، مقدار UiState به‌روزرسانی شده و Compose با حالت جدید رندر شود.

  

• **ترکیب با Jetpack Compose:**

هنگام رندر رابط کاربری با Compose، از Coroutineها (مثلاً در LaunchedEffect) برای فراخوانی غیرهمزمان و سپس await برای دریافت نتیجه استفاده کنید. این کار باعث می‌شود UI همیشه به‌روز و بدون بلاک شدن بماند.

  

**نتیجه‌گیری**

  

تابع await در Coroutineهای کاتلین ابزاری قدرتمند برای مدیریت همزمانی و عملیات غیرهمزمان است. با async می‌توانید چندین کار را به صورت موازی آغاز و با await نتایج را به صورت ترتیبی و غیربلاک‌کننده دریافت کنید. این رویکرد در ساخت اپلیکیشن‌های انعطاف‌پذیر، بهینه و پاسخ‌گو بسیار کارآمد است.

  

**نکات کلیدی:**

• await

یک تابع معلق است که نتیجه‌ی Deferred را بدون بلاک کردن رشته اصلی برمی‌گرداند.

• با async می‌توانید عملیات همزمان را مدیریت کرده و در نهایت با await نتیجه‌ها را بگیرید.

• در موقعیت‌های واقعی نظیر فراخوانی APIها، پردازش چندین کار موازی، یا به‌روز‌رسانی UI در معماری‌های مدرن MVI/Compose، await ابزار ارزشمندی به شمار می‌آید.

• با رعایت مدیریت خطا، الگوهای معماری صحیح، و استفاده از Scopeهای مناسب، می‌توانید از پتانسیل کامل await بهره‌مند شوید.

  

**منابع و مراجع**

  

[1] [Kotlin Deferred Await Documentation](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html)

[2] [DhiWise - Kotlin Deferred Await](https://www.dhiwise.com/post/kotlin-deferred-await-streamline-your-asynchronous-programming)

[3] [TechYourChance - Kotlin Coroutines vs Threads Performance Benchmark](https://www.techyourchance.com/kotlin-coroutines-vs-threads-performance-benchmark/)

[4] [GitHub: JetMVI-Template](https://github.com/myofficework000/JetMVI-Template)

[5] [DhiWise - Mastering Asynchronous Programming](https://www.dhiwise.com/post/mastering-asynchronous-programming-a-deep-dive)

[6] [ProAndroidDev - Awaiting Multiple Coroutines](https://proandroiddev.com/awaiting-multiple-coroutines-the-clean-way-75469f8df160)

[7] [StackOverflow: Difference between launch, join and async, await](https://stackoverflow.com/questions/46226518/what-is-the-difference-between-launch-join-and-async-await-in-kotlin-coroutines)

[8] [Kodeco (RayWenderlich): Kotlin Coroutines by Tutorials](https://www.kodeco.com/books/kotlin-coroutines-by-tutorials/v2.0/chapters/5-async-await)

[9] [Exploring Advanced MVI Pattern in Android App Development](https://blog.stackademic.com/exploring-advanced-mvi-pattern-in-android-app-development-d691a7dcafc9)

[10] [MVI Architecture in Compose Multiplatform](https://avwie.github.io/mvi-architecture-in-compose-multiplatform/)

[11] [Lankydan.dev - async-await in Coroutines](https://lankydan.dev/async-await-in-coroutines/)

[12] [Android Developer: Advanced Coroutines](https://developer.android.com/kotlin/coroutines/coroutines-adv)

[13] [StackOverflow: async vs launch](https://stackoverflow.com/questions/52820510/kotlin-async-vs-launch)

[14] [Reddit: C# async/await vs Kotlin async](https://www.reddit.com/r/Kotlin/comments/a5d7qq/c_asyncawait_vs_kotlin_async_coroutines_what_is/)