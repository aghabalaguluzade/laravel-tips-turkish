## Log and debug

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Factories)](factories.md) ➡️ [Next (API)](api.md)

- [Logging with parameters](#logging-with-parameters)
- [Uzun süre çalışan laravel sorgularını günlüğe kaydet](#uzun-süre-çalışan-laravel-sorgularını-günlüğe-kaydet)
- [Benchmark class](#benchmark-class)
- [Daha kullanışlı DD](#daha-kullanışlı-dd)
- [Log with context](#log-with-context)
- [Eloquent sorgusunu hızla SQL biçiminde çıktılayın](#eloquent-sorgusunu-hızla-sql-biçiminde-çıktılayın)
- [Geliştirme sırasındaki tüm veritabanı sorgularını günlüğe kaydedin](#geliştirme-sırasındaki-tüm-veritabanı-sorgularını-günlüğe-kaydedin)
- [Tek bir istekte ateşlenen tüm olayları keşfedin](#tek-bir-istekte-ateşlenen-tüm-olayları-keşfedin)

### Logging with parameters

Ne olduğu hakkında daha fazla bağlam için `Log::info()` veya ek parametrelerle daha kısa `info()` mesajı yazabilirsiniz.

```php
Log::info('User failed to login.', ['id' => $user->id]);
```

### Uzun süre çalışan laravel sorgularını günlüğe kaydet

```php
DB::enableQueryLog();

DB::whenQueryingForLongerThen(1000, function ($connection) {
     Log::warning(
          'Long running queries have been detected.',
          $connection->getQueryLog()
     );
});
```

Tip given by [@realstevebauman](https://twitter.com/realstevebauman/status/1576980397552185344)

### Benchmark class

Laravel 9.32'de herhangi bir görevin süresini ölçebilen bir Benchmark sınıfımız var.

Oldukça kullanışlı bir yardımcıdır:
```php
class OrderController
{
     public function index()
     {
          return Benchmark::measure(fn () => Order::all()),
     }
}
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1583096196494553088)

### Daha kullanışlı DD

`dd($result)` yapmak yerine, Eloquent cümlenizin veya herhangi bir Koleksiyonun sonuna doğrudan bir yöntem olarak `->dd()` koyabilirsiniz.

```php
// Yerine
$users = User::where('name', 'Taylor')->get();
dd($users);
// Bunu yap
$users = User::where('name', 'Taylor')->get()->dd();
```

### Log with context

Laravel 8.49'daki yenilik: `Log::withContext()`, farklı istekler arasındaki Log mesajlarını ayırt etmenize yardımcı olacaktır.

Bir Ara Yazılım oluşturur ve bu bağlamı ayarlarsanız, tüm Günlük (Log) mesajları bu bağlamı içerecek ve bunları daha kolay arayabileceksiniz.

```php
public function handle(Request $request, Closure $next)
{
    $requestId = (string) Str::uuid();

    Log::withContext(['request-id' => $requestId]);

    $response = $next($request);

    $response->header('request-id', $requestId);

    return $response;
}
```

### Eloquent sorgusunu hızla SQL biçiminde çıktılayın

Bir Eloquent sorgusunun SQL biçiminde hızlı bir şekilde çıktısını almak istiyorsanız, toSql() yöntemini bu şekilde çağırabilirsiniz.

```php
$invoices = Invoice::where('client', 'James pay')->toSql();

dd($invoices)
// select * from `invoices` where `client` = ?
```

Tip given by [@devThaer](https://twitter.com/devThaer/status/1438816135881822210)

### Geliştirme sırasındaki tüm veritabanı sorgularını günlüğe kaydedin

Geliştirme sırasında tüm veritabanı sorgularını günlüğe kaydetmek istiyorsanız AppServiceProvider'ınıza şu kod parçacığını ekleyin

```php
public function boot()
{
    if (App::environment('local')) {
        DB::listen(function ($query) {
            logger(Str::replaceArray('?', $query->bindings, $query->sql));
        });
    }
}
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1473262634405449730)

### Tek bir istekte ateşlenen tüm olayları keşfedin

Belirli bir olay için yeni bir dinleyici uygulamak istiyorsanız ancak adını bilmiyorsanız, istek sırasında ateşlenen tüm olayları günlüğe kaydedebilirsiniz.

Ateşlenen tüm olayları yakalamak için `app/Providers/EventServiceProvider.php`nin `boot()` yönteminde `\Illuminate\Support\Facades\Event::listen()` yöntemini kullanabilirsiniz.

**Önemli:** Bu olay dinleyicisi içinde `Log` cephesini kullanırsanız, sonsuz bir döngüden kaçınmak için `Illuminate\Log\Events\MessageLogged` adlı olayları hariç tutmanız gerekecektir. 
(Örnek: `if ($event == 'Illuminate\\Log\\Events\\MessageLogged') return;`)

```php
// Include Event...
use Illuminate\Support\Facades\Event;

// In your EventServiceProvider class...
public function boot()
{
    parent::boot();

    Event::listen('*', function ($event, array $data) {
        // Log the event class
        error_log($event);

        // Log the event data delegated to listener parameters
        error_log(json_encode($data, JSON_PRETTY_PRINT));
    });
}
```

Tip given by [@MuriloChianfa](https://github.com/MuriloChianfa)
