

# راهنمای جامع `CoroutineScope` در کوروتین‌های کاتلین

  

`CoroutineScope`

یکی از مفاهیم کلیدی در مدیریت کوروتین‌های کاتلین است که به سازماندهی و کنترل چرخه حیات آن‌ها کمک می‌کند. در این راهنما، به بررسی مفهوم، کاربردها، مثال‌های عملی، بهترین روش‌ها و اشتباهات رایج در استفاده از `CoroutineScope` می‌پردازیم.

  

---

  

## مفهوم `CoroutineScope`

  

`CoroutineScope`

یک رابط (interface) در کاتلین است که محدوده‌ای برای اجرای کوروتین‌ها تعریف می‌کند. این محدوده به مدیریت چرخه حیات کوروتین‌ها کمک کرده و از نشت حافظه جلوگیری می‌کند. هر سازنده کوروتین (مانند `launch` و `async`) یک اکستنشن بر روی `CoroutineScope` است و از `coroutineContext` آن به ارث می‌برد تا به‌صورت خودکار تمام المان‌ها و لغو (cancellation) را انتقال دهد. [oai_citation_attribution:4‡Kotlin](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-coroutine-scope/?utm_source=chatgpt.com)

  

### اهمیت در مدیریت چرخه حیات کوروتین‌ها

  

با استفاده از `CoroutineScope`، می‌توان اطمینان حاصل کرد که کوروتین‌ها در محدوده مشخصی اجرا می‌شوند و در صورت نیاز به‌صورت خودکار لغو می‌شوند. این امر به جلوگیری از نشت حافظه و مدیریت بهینه منابع کمک می‌کند.

  

### اجزای کلیدی `CoroutineScope`

  

- ****Job****:

- کنترل‌کننده چرخه حیات کوروتین که اطلاعاتی مانند تکمیل یا شکست آن را در بر می‌گیرد.

  

- ****CoroutineContext****:

- محیط اجرایی کوروتین را تعیین می‌کند، از جمله دیسپچری که کوروتین روی آن اجرا می‌شود.

  

در کاتلین، هر `CoroutineScope` دارای یک `Job` و یک `CoroutineContext` است که مدیریت و کنترل کوروتین‌ها را امکان‌پذیر می‌سازد. [oai_citation_attribution:3‡Sling Academy](https://www.slingacademy.com/article/how-to-use-coroutinescope-in-kotlin-for-cleaner-code/?utm_source=chatgpt.com)

  

---

  

## سناریوهای کاربردی

  

### استفاده در ViewModel در اندروید (مانند `viewModelScope`)

  

در اندروید، هر ViewModel دارای یک `viewModelScope` است که کوروتین‌های راه‌اندازی‌شده در این محدوده به‌صورت خودکار در صورت پاک شدن ViewModel لغو می‌شوند. این ویژگی برای انجام کارهایی که فقط در زمان فعال بودن ViewModel نیاز به اجرا دارند، مفید است. [oai_citation_attribution:2‡Android Developers](https://developer.android.com/topic/libraries/architecture/coroutines?utm_source=chatgpt.com)

  

### مدیریت کوروتین‌ها با آگاهی از چرخه حیات در لایه‌های UI

  

با استفاده از `LifecycleScope`، می‌توان کوروتین‌هایی را راه‌اندازی کرد که با چرخه حیات کامپوننت‌های UI هماهنگ باشند و در زمان تخریب آن‌ها به‌صورت خودکار لغو شوند. [oai_citation_attribution:1‡Android Developers](https://developer.android.com/topic/libraries/architecture/coroutines?utm_source=chatgpt.com)

  

### ساختاردهی به کوروتین‌ها در وظایف پس‌زمینه یا لایه‌های مخزن (Repository)

  

در لایه‌های مخزن، می‌توان از `CoroutineScope` برای مدیریت وظایف پس‌زمینه مانند درخواست‌های شبکه یا عملیات دیتابیس استفاده کرد و اطمینان حاصل کرد که این وظایف به‌درستی مدیریت و در صورت نیاز لغو می‌شوند.

  

---

  

## مثال‌های عملی

  

### ایجاد یک `CoroutineScope` سفارشی با کانتکست یا دیسپچر مشخص

  

```kotlin

class MyRepository {

    private val scope = CoroutineScope(Job() + Dispatchers.IO)

  

    fun fetchData() {

        scope.launch {

            // اجرای عملیات I/O

        }

    }

  

    fun clear() {

        scope.cancel() // لغو تمام کوروتین‌های مرتبط

    }

}

```
  

**توضیح**: در این مثال، یک CoroutineScope با یک Job و Dispatchers.IO ایجاد شده است. تابع fetchData یک کوروتین را در این محدوده راه‌اندازی می‌کند و تابع clear تمام کوروتین‌های مرتبط را لغو می‌کند.

  

**مدیریت چندین کوروتین در یک محدوده**

  
```

fun main() = runBlocking {

    val scope = CoroutineScope(Dispatchers.Default)

  

    scope.launch {

        _// کوروتین اول_

    }

  

    scope.launch {

        _// کوروتین دوم_

    }

  

    delay(1000)

    scope.cancel() _// لغو تمام کوروتین‌های مرتبط_

}

  ```

**توضیح**: در این مثال، دو کوروتین در یک CoroutineScope راه‌اندازی شده‌اند. پس از یک ثانیه، هر دو کوروتین لغو می‌شوند.

  

**لغو یک** **CoroutineScope** **و کوروتین‌های مرتبط**

  ```


class MyViewModel : ViewModel() {

    private val scope = CoroutineScope(Job() + Dispatchers.Main)

  

    fun loadData() {

        scope.launch {

            _// بارگذاری داده‌ها_

        }

    }

  

    override fun onCleared() {

        super.onCleared()

        scope.cancel() _// لغو تمام کوروتین‌های مرتبط_

    }

}

  

```
**توضیح**: در این مثال، یک CoroutineScope در ViewModel ایجاد شده است. هنگام پاک شدن ViewModel، تمام کوروتین‌های مرتبط لغو می‌شوند.

  

**بهترین روش‌ها و اشتباهات رایج**

  

**بهترین روش‌ها**

• **اجتناب از استفاده از** GlobalScope **برای همزمانی ساختاری**: استفاده از GlobalScope می‌تواند منجر به نشت حافظه شود؛ بنابراین، از محدوده‌های محلی استفاده کنید.

• **لغو صحیح محدوده‌ها برای پاک‌سازی منابع**: اطمینان حاصل کنید که محدوده‌ها در زمان مناسب لغو می‌شوند تا منابع آزاد شوند.

  

**اشتباهات رایج**

• **عدم مدیریت صحیح چرخه حیات کوروتین‌ها**: عدم لغو کوروتین‌ها می‌تواند منجر به نشت حافظه شود.

• **استفاده نادرست از دیسپچرها**: اجرای عملیات سنگین روی Dispatchers.Main می‌تواند باعث کاهش عملکرد شود.

  

**منابع و مستندات**

• [CoroutineScope - مستندات رسمی کاتلین](https://k