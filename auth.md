## Auth

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Collections)](collections.md) ➡️ [Next (Mail)](mail.md)

- [Aynı anda birden fazla izni kontrol edin](#aynı-anda-birden-fazla-izni-kontrol-edin)
- [Kullanıcıları daha fazla seçenekle doğrulayın](#kullanıcıları-daha-fazla-seçenekle-doğrulayın)
- [Kullanıcı kaydıyla ilgili daha fazla etkinlik (events)](#kullanıcı-kaydıyla-ilgili-daha-fazla-etkinlik-events)
- [Auth::once()'u biliyor muydunuz?](#authonceu-biliyor-muydunuz)
- [Kullanıcı şifre güncellemesinde API Token'ı değiştirme](#kullanıcı-şifre-güncellemesinde-api-tokenı-değiştirme)
- [Süper yönetici için izinleri geçersiz kılma](#süper-yönetici-için-izinleri-geçersiz-kılma)

### Aynı anda birden fazla izni kontrol edin

`@can` Blade direktifinin yanı sıra, `@canany` direktifiyle birden fazla izni aynı anda kontrol edebileceğinizi biliyor muydunuz?

```blade
@canany(['update', 'view', 'delete'], $post)
    // Geçerli kullanıcı yayını güncelleyebilir, görüntüleyebilir veya silebilir
@elsecanany(['create'], \App\Post::class)
    // Geçerli kullanıcı gönderi oluşturabilir
@endcanany
```

### Kullanıcıları daha fazla seçenekle doğrulayın

Örneğin, yalnızca "etkinleştirilmiş" kullanıcıların kimliğini doğrulamak istiyorsanız, bu, "Auth::attempt()"'a fazladan bir argüman iletmek kadar basittir.

Karmaşık ara katman yazılımlarına veya global kapsamlara gerek yok.

```php
Auth::attempt(
    [
        ...$request->only('email', 'password'),
        fn ($query) => $query->whereNotNull('activated_at')
    ],
    $this->boolean('remember')
);
```

Tip given by [@LukeDowning19](https://twitter.com/LukeDowning19)

### Kullanıcı kaydıyla ilgili daha fazla etkinlik (events)

Yeni kullanıcı kaydından sonra bazı işlemler mi gerçekleştirmek istiyorsunuz? 'app/Providers/EventServiceProvider.php' adresine gidin ve daha fazla Dinleyici (Listeners) sınıfı ekleyin ve ardından bu sınıflarda '$event->user' nesnesiyle 'handle()' yöntemini uygulayın.

```php
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        Registered::class => [
            SendEmailVerificationNotification::class,

            // Buraya herhangi bir Dinleyici sınıfını ekleyebilirsiniz
            // Bu sınıfın içindeki handle() yöntemi ile
        ],
    ];
```

### Auth::once()'u biliyor muydunuz?

Kullanıcıyla yalnızca BİR İSTEK için, 'Auth::once()' yöntemini kullanarak giriş yapabilirsiniz.
Hiçbir oturum veya çerez kullanılmayacaktır, bu da bu yöntemin durum bilgisi olmayan bir API (stateless API) oluştururken yararlı olabileceği anlamına gelir.

```php
if (Auth::once($credentials)) {
    //
}
```

### Kullanıcı şifre güncellemesinde API Token'ı değiştirme

Parolası değiştiğinde kullanıcının API Token'ını değiştirmek pratik olacaktır.

Model:

```php
protected function password(): Attribute
{
    return Attribute::make(
            set: function ($value, $attributes) {
                $value = $value;
                $attributes['api_token'] = Str::random(100);
            }
        );
}
```

### Süper yönetici için izinleri geçersiz kılma

Gates'inizi tanımladıysanız ancak SUPER ADMIN kullanıcısı için tüm izinleri geçersiz kılmak istiyorsanız, bu süper yöneticiye TÜM izinleri vermek için, `AuthServiceProvider.php` dosyasındaki `Gate::before()` deyimiyle geçitleri durdurabilirsiniz.

```php
// Herhangi bir Gate durdurun ve süper yönetici olup olmadığını kontrol edin
Gate::before(function($user, $ability) {
    if ($user->is_super_admin == 1) {
        return true;
    }
});

// Ya da bazı izin paketleri kullanırsanız...
Gate::before(function($user, $ability) {
    if ($user->hasPermission('root')) {
        return true;
    }
});
```

Eğer Gate'inizde hiç kullanıcı yokken bir şey yapmak istiyorsanız, '$user' için 'null' olmasına izin veren bir tür ipucu eklemeniz gerekir. Örneğin, oturum açmamış kullanıcılarınız için Anonim adında bir rolünüz varsa:

```php
Gate::before(function (?User $user, $ability) {
    if ($user === null) {
        $role = Role::findByName('Anonymous');
        return $role->hasPermissionTo($ability) ? true : null;
    }
    return $user->hasRole('Super Admin') ? true : null;
});
```
