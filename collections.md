## Collections

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Validation)](validation.md) ➡️ [Next (Auth)](auth.md)

- [Use groupBy on Collections with Custom Callback Function](#use-groupby-on-collections-with-custom-callback-function)
- [Laravel Scopes can be combined using "Higher Order" orWhere Method](#laravel-scopes-can-be-combined-using-higher-order-orwhere-method)
- [Tek satırda çoklu Collection yöntemleri](#tek-satırda-çoklu-collection-yöntemleri)
- [Sayfalandırmayla Toplamı Hesapla](#sayfalandırmayla-toplamı-hesapla)
- [Sayfalandırmalı foreach döngüsündeki seri numarası](#sayfalandırmalı-foreach-döngüsündeki-seri-numarası)
- [Higher order collection message](#higher-order-collection-message)
- [Mevcut bir anahtarı alın veya mevcut değilse bir değer ekleyin ve değeri döndürün](#mevcut-bir-anahtarı-alın-veya-mevcut-değilse-bir-değer-ekleyin-ve-değeri-döndürün)
- [Static times method](#static-times-method)

### Use groupBy on Collections with Custom Callback Function

Sonuçları veritabanınızda doğrudan bir sütun olmayan bir koşula göre gruplamak istiyorsanız, bunu bir kapatma işlevi sağlayarak yapabilirsiniz.

Örneğin, kullanıcıları kayıt tarihine göre gruplandırmak istiyorsanız kod şu şekildedir:

```php
$users = User::all()->groupBy(function($item) {
    return $item->created_at->format('Y-m-d');
});
```

⚠️ Dikkat: bu işlem bir `Collection` sınıfı üzerinde yapılır, yani sonuçlar veritabanından alındıktan ** SONRA** gerçekleştirilir.

### Laravel Scopes can be combined using "Higher Order" orWhere Method

Dokümanlar'dan aşağıdaki örnek.

Önce:
```php
User::popular()->orWhere(function (Builder $query) {
     $query->active();
})->get()
```

Sonra:
```php
User::popular()->orWhere->active()->get();
```

Tip given by [@TheLaravelDev](https://twitter.com/TheLaravelDev/status/1564608208102199298/)

### Tek satırda çoklu Collection yöntemleri

Tüm sonuçları `->all()` veya `->get()` ile sorgularsanız, daha sonra aynı sonuç üzerinde çeşitli Collection işlemleri gerçekleştirebilirsiniz, her seferinde veritabanını sorgulamaz.

```php
$users = User::all();
echo 'Max ID: ' . $users->max('id');
echo 'Average age: ' . $users->avg('age');
echo 'Total budget: ' . $users->sum('budget');
```

### Sayfalandırmayla Toplamı Hesapla

Yalnızca PAGINATED koleksiyonuna sahip olduğunuzda tüm kayıtların toplamı nasıl hesaplanır? Hesaplamayı sayfalandırmadan ÖNCE, ancak aynı sorgudan yapın.

```php
// Sayfalandırmayla post_view'lerin toplamı nasıl alınır?
$posts = Post::paginate(10);
// Bu sadece 1. sayfa için olacak, TÜM gönderiler için değil
$sum = $posts->sum('post_views');

// Bunu Query Builder ile yapın
$query = Post::query();
// Toplamı hesapla
$sum = $query->sum('post_views');
// Ve sonra aynı sorgudan sayfalandırmayı yapın
$posts = $query->paginate(10);
```

### Sayfalandırmalı foreach döngüsündeki seri numarası

Sayfalandırmada foreach koleksiyon öğeleri indeksini seri no (SL) olarak kullanabiliriz.

```php
   ...
   <th>Serial</th>
    ...
    @foreach ($products as $product)
    <tr>
        <td>{{ $loop->index + $product->firstItem() }}</td>
        ...
    @endforeach
```

sonraki sayfalar(?page=2&...) dizin sayısının devam etmesi sorununu çözecektir.

### Higher order collection message

Koleksiyonlar ayrıca, koleksiyonlar üzerinde ortak eylemler gerçekleştirmek için kısa yollar olan "higher order messages" için destek sağlar.
Bu örnek, bir teklifteki ürün grubu başına fiyatı hesaplar.

```php
$offer = [
        'name'  => 'offer1',
        'lines' => [
            ['group' => 1, 'price' => 10],
            ['group' => 1, 'price' => 20],
            ['group' => 2, 'price' => 30],
            ['group' => 2, 'price' => 40],
            ['group' => 3, 'price' => 50],
            ['group' => 3, 'price' => 60]
        ]
];

$totalPerGroup = collect($offer['lines'])->groupBy->group->map->sum('price');
```

### Mevcut bir anahtarı alın veya mevcut değilse bir değer ekleyin ve değeri döndürün

Laravel 8.81`de Collections`a `getOrPut` metodu, mevcut bir anahtarı almak veya mevcut değilse bir değer eklemek ve değeri döndürmek istediğiniz kullanım durumunu basitleştirir.

```php
$key = 'name';
// Still valid
if ($this->collection->has($key) === false) {
    $this->collection->put($key, ...);
}

return $this->collection->get($key);

// Using the `getOrPut()` method with closure
return $this->collection->getOrPut($key, fn() => ...);

// Or pass a fixed value
return $this->collection->getOrPut($key, $value='teacoders');
```

Tip given by [@Teacoders](https://twitter.com/Teacoders/status/1488338815592718336)

### Static times method

Statik times yöntemi, verilen closure'ı belirtilen sayıda çağırarak yeni bir koleksiyon oluşturur.

```php
Collection::times(7, function ($number) {
    return now()->addDays($number)->format('d-m-Y');
});
// Output: [01-04-2022, 02-04-2022, ..., 07-04-2022]
```

Tip given by [@Teacoders](https://twitter.com/Teacoders/status/1509447909602906116)

