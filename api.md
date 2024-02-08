## API

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Log and debug)](log-and-debug.md) ➡️ [Next (Other)](other.md)

- [API Resources: With or Without "data"?](#api-resources-with-or-without-data)
- [API kaynaklarında koşullu ilişki sayıları](#apı-kaynaklarında-koşullu-ilişki-sayıları)
- [API Return "Everything went ok"](#api-return-everything-went-ok)
- [API kaynaklarında N+1 sorgudan kaçının](#aPI-kaynaklarında-N+1-sorgudan-kaçının)
- [Get Bearer Token from Authorization header](#get-bearer-token-from-authorization-header)
- [API sonuçlarınızı sıralama](#apı-sonuçlarınızı-sıralama)
- [Customize Exception Handler For API](#customize-exception-handler-for-api)
- [Force JSON Response For API Requests](#force-json-response-for-api-requests)
- [API Versioning](#api-versioning)

### API Resources: With or Without "data"?

Veri döndürmek için Eloquent API Kaynakları kullanırsanız, bunlar otomatik olarak 'data' ile sarılacaktır. Bunu kaldırmak istiyorsanız, `app/Providers/AppServiceProvider.php` dosyasına `JsonResource::withoutWrapping();` ekleyin.

```php
class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        JsonResource::withoutWrapping();
    }
}
```

Tip given by [@phillipmwaniki](https://twitter.com/phillipmwaniki/status/1445230637544321029)

### API kaynaklarında koşullu ilişki sayıları

whenCounted yöntemini kullanarak bir ilişkinin sayımını kaynak yanıtınıza koşullu olarak dahil edebilirsiniz. Bunu yaptığınızda, ilişkilerin sayısı eksikse öznitelik dahil edilmez.
```php
public function toArray($request)
{
     return [
          'id' => $this->id,
          'name' => $this->name,
          'email' => $this->email,
          'posts_count' => $this->whenCounted('posts'),
          'created_at' => $this->created_at,
          'updated_at' => $this->updated_at,
     ];
}
```

Tip given by [@mvpopuk](https://twitter.com/mvpopuk/status/1570480977507504128)

### API Return "Everything went ok"

Bazı işlemleri gerçekleştiren ancak yanıt vermeyen API uç noktanız varsa ve bu nedenle yalnızca "her şey yolunda gitti" ifadesini döndürmek istiyorsanız, 204 durum kodunu "İçerik yok" olarak döndürebilirsiniz. Laravel'de bu kolaydır: `return Response()->noContent();`.

```php
public function reorder(Request $request)
{
    foreach ($request->input('rows', []) as $row) {
        Country::find($row['id'])->update(['position' => $row['position']]);
    }

    return response()->noContent();
}
```

### API kaynaklarında N+1 sorgudan kaçının

API kaynaklarında `whenLoaded()` yöntemini kullanarak N+1 sorgularından kaçınabilirsiniz.

Bu, departmanı yalnızca Çalışan modelinde zaten yüklüyse ekleyecektir.

`whenLoaded()` olmadan departman için her zaman bir sorgu vardır

```php
class EmployeeResource extends JsonResource
{
    public function toArray($request): array
    {
        return [
            'id' => $this->uuid,
            'fullName' => $this->full_name,
            'email' => $this->email,
            'jobTitle' => $this->job_title,
            'department' => DepartmentResource::make($this->whenLoaded('department')),
        ];
    }
}
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1473987501501071362)

### Get Bearer Token from Authorization header

Apiler ile çalışırken ve Authorization başlığından token'a erişmek istediğinizde `bearerToken()` fonksiyonu çok kullanışlıdır.

```php
// API başlıklarını bu şekilde manuel olarak ayrıştırmayın:
$tokenWithBearer = $request->header('Authorization');
$token = substr($tokenWithBearer, 7);

//Bunun yerine şunu yapın:
$token = $request->bearerToken();
```

Tip given by [@iamharis010](https://twitter.com/iamharis010/status/1488413755826327553)

### API sonuçlarınızı sıralama

Single-column API sorting, with direction control

```php
// Handles /dogs?sort=name and /dogs?sort=-name
Route::get('dogs', function (Request $request) {
    // Get the sort query parameter (or fall back to default sort "name")
    $sortColumn = $request->input('sort', 'name');

    // Set the sort direction based on whether the key starts with -
    // using Laravel's Str::startsWith() helper function
    $sortDirection = Str::startsWith($sortColumn, '-') ? 'desc' : 'asc';
    $sortColumn = ltrim($sortColumn, '-');

    return Dog::orderBy($sortColumn, $sortDirection)
        ->paginate(20);
});
```

we do the same for multiple columns (e.g., ?sort=name,-weight)

```php
// Handles ?sort=name,-weight
Route::get('dogs', function (Request $request) {
    // Grab the query parameter and turn it into an array exploded by ,
    $sorts = explode(',', $request->input('sort', ''));

    // Create a query
    $query = Dog::query();

    // Add the sorts one by one
    foreach ($sorts as $sortColumn) {
        $sortDirection = Str::startsWith($sortColumn, '-') ? 'desc' : 'asc';
        $sortColumn = ltrim($sortColumn, '-');

        $query->orderBy($sortColumn, $sortDirection);
    }

    // Return
    return $query->paginate(20);
});
```
---

### Customize Exception Handler For API

#### Laravel 8 and below:

There's a method `render()` in `App\Exceptions` class:

```php
   public function render($request, Exception $exception)
    {
        if ($request->wantsJson() || $request->is('api/*')) {
            if ($exception instanceof ModelNotFoundException) {
                return response()->json(['message' => 'Item Not Found'], 404);
            }

            if ($exception instanceof AuthenticationException) {
                return response()->json(['message' => 'unAuthenticated'], 401);
            }

            if ($exception instanceof ValidationException) {
                return response()->json(['message' => 'UnprocessableEntity', 'errors' => []], 422);
            }

            if ($exception instanceof NotFoundHttpException) {
                return response()->json(['message' => 'The requested link does not exist'], 400);
            }
        }

        return parent::render($request, $exception);
    }
```

#### Laravel 9 and above:

There's a method `register()` in `App\Exceptions` class:

```php
    public function register()
    {
        $this->renderable(function (ModelNotFoundException $e, $request) {
            if ($request->wantsJson() || $request->is('api/*')) {
                return response()->json(['message' => 'Item Not Found'], 404);
            }
        });

        $this->renderable(function (AuthenticationException $e, $request) {
            if ($request->wantsJson() || $request->is('api/*')) {
                return response()->json(['message' => 'unAuthenticated'], 401);
            }
        });
        $this->renderable(function (ValidationException $e, $request) {
            if ($request->wantsJson() || $request->is('api/*')) {
                return response()->json(['message' => 'UnprocessableEntity', 'errors' => []], 422);
            }
        });
        $this->renderable(function (NotFoundHttpException $e, $request) {
            if ($request->wantsJson() || $request->is('api/*')) {
                return response()->json(['message' => 'The requested link does not exist'], 400);
            }
        });
    }
```

Tip given by [Feras Elsharif](https://github.com/ferasbbm)

---

### Force JSON Response For API Requests

Bir API oluşturduysanız ve istek "Accept: application/JSON " HTTP Başlığı içermediğinde bir hatayla karşılaşırsa, hata API rotalarında HTML veya yönlendirme yanıtı olarak döndürülür, bu nedenle bundan kaçınmak için tüm API yanıtlarını JSON'a zorlayabiliriz.

İlk adım, bu komutu çalıştırarak ara yazılım oluşturmaktır:

```console
php artisan make:middleware ForceJsonResponse
```

Bu kodu `App/Http/Middleware/ForceJsonResponse.php` dosyasındaki handle fonksiyonuna yazın:

```php
public function handle($request, Closure $next)
{
    $request->headers->set('Accept', 'application/json');
    return $next($request);
}
```

İkinci olarak, oluşturulan ara yazılımı app/Http/Kernel.php dosyasına kaydedin:

```php
protected $middlewareGroups = [        
    'api' => [
        \App\Http\Middleware\ForceJsonResponse::class,
    ],
];
```

Tip given by [Feras Elsharif](https://github.com/ferasbbm)

---

### API Versioning

#### When to version?

Gelecekte çoklu sürüme sahip olabilecek bir proje üzerinde çalışıyorsanız veya uç noktalarınızda yanıt verilerinin biçimindeki değişiklik gibi önemli bir değişiklik varsa ve API sürümünün, değişiklik yapıldığında işlevsel kalmasını sağlamak istiyorsanız kod.

#### Change The Default Route Files 
İlk adım, `App\Providers\RouteServiceProvider` dosyasındaki rota haritasını değiştirmektir, o halde başlayalım:

#### Laravel 8 and above:

Add a 'ApiNamespace' property 

```php
/**
 * @var string
 *
 */
protected string $ApiNamespace = 'App\Http\Controllers\Api';
```

Inside the method boot, add the following code:

```php
$this->routes(function () {
     Route::prefix('api/v1')
        ->middleware('api')
        ->namespace($this->ApiNamespace.'\\V1')
        ->group(base_path('routes/API/v1.php'));
        }
    
    //for v2
     Route::prefix('api/v2')
            ->middleware('api')
            ->namespace($this->ApiNamespace.'\\V2')
            ->group(base_path('routes/API/v2.php'));
});
```


#### Laravel 7 and below:

Add a 'ApiNamespace' property

```php
/**
 * @var string
 *
 */
protected string $ApiNamespace = 'App\Http\Controllers\Api';
```

Inside the method map, add the following code:

```php
// remove this $this->mapApiRoutes(); 
    $this->mapApiV1Routes();
    $this->mapApiV2Routes();
```

And add these methods:

```php
  protected function mapApiV1Routes()
    {
        Route::prefix('api/v1')
            ->middleware('api')
            ->namespace($this->ApiNamespace.'\\V1')
            ->group(base_path('routes/Api/v1.php'));
    }

  protected function mapApiV2Routes()
    {
        Route::prefix('api/v2')
            ->middleware('api')
            ->namespace($this->ApiNamespace.'\\V2')
            ->group(base_path('routes/Api/v2.php'));
    }
```

#### Controller Folder Versioning

```
Controllers
└── Api
    ├── V1
    │   └──AuthController.php
    └── V2
        └──AuthController.php
```

#### Route File Versioning

```
routes
└── Api
   │    └── v1.php     
   │    └── v2.php 
   └── web.php
```

Tip given by [Feras Elsharif](https://github.com/ferasbbm)
