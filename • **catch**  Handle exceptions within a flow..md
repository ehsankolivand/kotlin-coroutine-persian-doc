# آموزش Catch در Kotlin Flow

## مقدمه
عملگر catch در Kotlin Flow یک ابزار قدرتمند برای مدیریت خطاها در جریان داده‌ها است. این عملگر به ما اجازه می‌دهد تا خطاهای رخ داده در بالادست (upstream) را مدیریت کنیم.

## اصول اساسی
1. **شفافیت خطا (Exception Transparency)**
   - فقط خطاهای بالادست را می‌گیرد
   - نسبت به خطاهای پایین‌دست شفاف است
   - نمی‌توان مستقیماً در بلوک try/catch داخل سازنده flow مقداری emit کرد

2. **خصوصیات رفتاری**
   - مانند پیچیدن کد بالادست در بلوک try/catch عمل می‌کند
   - امکان emit مقادیر جدید از بلوک catch را فراهم می‌کند
   - می‌تواند خطاها را مجدداً پرتاب یا تبدیل کند

## نمونه کد پایه
```kotlin
flow {
    emit(fetchData()) // عملیات بالادست
}
.catch { exception -> 
    emit(fallbackData) // emit مقدار جایگزین
}
.collect { data ->
    processData(data) // عملیات پایین‌دست
}
```

## الگوهای پیاده‌سازی پیشرفته
### مدیریت وضعیت با StateFlow
```kotlin
private val _stateFlow = MutableStateFlow<UiState>(UiState.Loading)
val stateFlow = _stateFlow.asStateFlow()

someFlow
    .catch { exception ->
        _stateFlow.value = UiState.Error(exception)
    }
    .collect { data ->
        _stateFlow.value = UiState.Success(data)
    }
```

## اشتباهات رایج
1. **جایگذاری نادرست عملگر**
   - catch باید قبل از عملگرهایی که نیاز به محافظت در برابر خطا دارند قرار گیرد
   - عملگرهای پایین‌دست توسط catch محافظت نمی‌شوند

2. **نقض شفافیت خطا**
```kotlin
// نادرست
flow {
    try {
        emit(data)
    } catch (e: Exception) {
        emit(fallback) // نقض شفافیت خطا
    }
}

// درست
flow { emit(data) }
    .catch { e -> emit(fallback) }
```

## بهترین شیوه‌های استفاده
1. **گرانولاریتی خطا**
   - به جای کلاس Exception عمومی، خطاهای خاص را catch کنید
   - فقط خطاهای قابل بازیابی را در بلوک‌های catch مدیریت کنید

2. **حفظ Context**
```kotlin
flow
    .flowOn(Dispatchers.IO)
    .catch { /* مدیریت خطا */ }
    .flowOn(Dispatchers.Main)
```

این مقاله به شما کمک می‌کند تا با مفاهیم اساسی و پیشرفته استفاده از catch در Kotlin Flow آشنا شوید و بتوانید خطاها را به شکل مؤثری در برنامه‌های خود مدیریت کنید.