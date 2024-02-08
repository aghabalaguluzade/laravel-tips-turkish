## Artisan

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Mail)](mail.md) ➡️ [Next (Factories)](factories.md)

- [Artisan command parameters](#artisan-command-parameters)
- [Komut hatasız çalıştıktan veya hata verdikten sonra bir closure çalıştırma](#komut-hatasız-çalıştıktan-veya-hata-verdikten-sonra-bir-closure-çalıştırma)
- [Belirli ortamlarda artisan komutlarını çalıştırma](#belirli-ortamlarda-artisan-komutlarını-çalıştırma)
- [Maintenance Mode](#maintenance-mode)
- [Artisan command help](#artisan-command-help)
- [Exact Laravel version](#exact-laravel-version)
- [Artisan komutunu her yerden başlatın](#artisan-komutunu-her-yerden-başlatın)
- [Özel komutunuzu gizleyin](#özel-komutunuzu-gizleyin)
- [Skip method](#skip-method)

### Artisan command parameters

Artisan komutunu oluştururken girişi çeşitli şekillerde sorabilirsiniz: `$this->confirm()`, `$this->anticipate()`, `$this->choice()`.

```php
// Yes or no?
if ($this->confirm('Do you wish to continue?')) {
    //
}

// Soruyu otomatik tamamlama seçenekleriyle açın
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);

// Varsayılan indeks ile listelenen seçeneklerden biri
$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
```

### Komut hatasız çalıştıktan veya hata verdikten sonra bir closure çalıştırma

Laravel scheduler ile, bir komut onSuccess() metodu ile hatasız çalıştığında ve ayrıca bir komut onFailure() metodu ile herhangi bir hata olduğunda bir Closure çalıştırabilirsiniz.

```php
protected function schedule(Schedule $schedule)
{
    $schedule->command('newsletter:send')
        ->mondays()
        ->onSuccess(fn () => resolve(SendNewsletterSlackNotification::class)->handle(true))
        ->onFailure(fn () => resolve(SendNewsletterSlackNotification::class)->handle(false));
}
```

Tip given by [@wendell_adriel](https://twitter.com/wendell_adriel)

### Belirli ortamlarda artisan komutlarını çalıştırma

Laravel zamanlanmış komutlarınızın kontrolünü elinize alın. En üst düzey esneklik için bunları belirli ortamlarda çalıştırın.

```php
$schedule->command('reports:send')
    ->daily()
    ->environments(['production', 'staging']);
```

Tip given by [@LaraShout](https://twitter.com/LaraShout)

### Maintenance Mode

Sayfanızda bakım modunu etkinleştirmek istiyorsanız down Artisan komutunu çalıştırın:

```bash
php artisan down
```

İnsanlar varsayılan 503 durum sayfasını görürler.

Ayrıca Laravel 8'de bayraklar da sağlayabilirsiniz:

- kullanıcının yönlendirilmesi gereken yol
- önceden oluşturulması gereken görünüm
- bakım modunu atlamak için gizli ifade
- her X saniyede bir sayfayı yeniden yüklemeyi deneyin

```bash
php artisan down --redirect="/" --render="errors::503" --secret="1630542a-246b-4b66-afa1-dd72a4c43515" --retry=60
```

Laravel 8'den önce:

- gösterilecek mesaj
- her X saniyede bir sayfayı yeniden yüklemeyi deneyin
- yine de bazı IP adreslerine erişime izin ver

```bash
php artisan down --message="Upgrading Database" --retry=60 --allow=127.0.0.1
```

Bakım işini tamamladığınızda şunu çalıştırın:

```bash
php artisan up
```

### Artisan command help

Artisan komutunun seçeneklerini kontrol etmek için artisan komutlarını `--help` bayrağıyla çalıştırın. Örneğin, 'php artisan make:model --help' ve kaç seçeneğiniz olduğunu görün:

```
Options:
  -a, --all             Generate a migration, seeder, factory, policy, resource controller, and form request classes for the model
  -c, --controller      Create a new controller for the model
  -f, --factory         Create a new factory for the model
      --force           Model zaten mevcut olsa bile sınıfı oluşturun
  -m, --migration       Create a new migration file for the model
      --morph-pivot     Oluşturulan modelin özel bir polimorfik ara tablo modeli olup olmayacağını belirtir
      --policy          Create a new policy for the model
  -s, --seed            Create a new seeder for the model
  -p, --pivot           Oluşturulan modelin özel bir ara tablo modeli olup olmayacağını belirtir
  -r, --resource        Oluşturulan denetleyicinin (controller) bir kaynak denetleyicisi olup olmayacağını belirtir
      --api             Oluşturulan denetleyicinin bir API kaynak denetleyicisi olup olmayacağını belirtir
  -R, --requests        Yeni form isteği sınıfları oluşturun ve bunları kaynak denetleyicisinde kullanın
      --test            Model için eşlik eden bir PHPUnit testi oluşturun
      --pest            Model için eşlik eden bir Pest testi oluşturun
  -h, --help            Verilen komut için yardımı görüntüler. Komut verilmediğinde liste komutu için yardımı görüntüler
  -q, --quiet           Herhangi bir mesaj çıkarmayın
  -V, --version         Display this application version
      --ansi|--no-ansi  Force (or disable --no-ansi) ANSI output
  -n, --no-interaction  İnteraktif soru sormayın
      --env[=ENV]       Komutun altında çalışması gereken ortam
  -v|vv|vvv, --verbose  Mesajların ayrıntı düzeyini artırın: Normal çıktı için 1, daha ayrıntılı çıktı için 2 ve hata ayıklama için 3
```

### Exact Laravel version

Komutu çalıştırarak uygulamanızda tam olarak hangi Laravel sürümüne sahip olduğunuzu öğrenin
`php artisan --version`

### Artisan komutunu her yerden başlatın

Eğer Artisan komutunuz varsa onu sadece Terminalden değil kodunuzun herhangi bir yerinden parametrelerle başlatabilirsiniz. Artisan::call() yöntemini kullanın:

```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
});
```

### Özel komutunuzu gizleyin

Artisan komut listesinde belirli bir komutun gösterilmesini istemiyorsanız, `hidden` özelliğini `true` olarak ayarlayın.

```php
class SendMail extends Command
{
    protected $signature = 'send:mail';
    protected $hidden = true;
}
```

Eğer `php artisan` yazdıysanız mevcut komutlar arasında `send:mail` göremezsiniz

Tip given by [@sky_0xs](https://twitter.com/sky_0xs/status/1487921500023832579)

### Skip method

Laravel zamanlayıcıda skip yöntemi

Bir yürütmeyi atlamak için komutlarınızda `skip` kullanabilirsiniz

```php
$schedule->command('emails:send')->daily()->skip(function () {
    return Calendar::isHoliday();
});
```

Tip given by [@cosmeescobedo](https://twitter.com/cosmeescobedo/status/1494503181438492675)

