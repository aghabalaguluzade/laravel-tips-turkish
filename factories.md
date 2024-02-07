## Factories

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Artisan)](artisan.md) ➡️ [Next (Log and debug)](log-and-debug.md)

- [Factory callbacks](#factory-callbacks)
- [Generate Images with Seeds/Factories](#generate-images-with-seedsfactories)
- [Değerleri geçersiz kılma ve onlara özel giriş uygulama](#değerleri-geçersiz-kılma-ve-onlara-özel-giriş-uygulama)
- [Using factories with relationships](#using-factories-with-relationships)
- [Herhangi bir olay göndermeden modeller oluşturun](#herhangi-bir-olay-göndermeden-modeller-oluşturun)
- [Useful for() method](#useful-for-method)
- [Run factories without dispatching events](#run-factories-without-dispatching-events)
- [run() yönteminde bağımlılıkları belirtin](#run-yönteminde-bağımlılıkları-belirtin)

### Factory callbacks

Veri ekleme için fabrikaları kullanırken, kayıt eklendikten sonra bazı eylemleri gerçekleştirmek için Fabrika Geri Çağırma (Factory Callback) işlevleri sağlayabilirsiniz.

```php
$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
```

### Generate Images with Seeds/Factories

Did you know that Faker can generate not only text values but also IMAGES? See `avatar` field here - it will generate 50x50 image:
Faker'ın yalnızca metin değerleri değil, aynı zamanda GÖRÜNTÜLER de oluşturabildiğini biliyor muydunuz? Kodda `avatar` alanına bakın - 50x50 resim oluşturacaktır:

```php
$factory->define(User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => bcrypt('password'),
        'remember_token' => Str::random(10),
        'avatar' => $faker->image(storage_path('images'), 50, 50)
    ];
});
```

### Değerleri geçersiz kılma ve onlara özel giriş uygulama

Factories ile kayıt oluştururken, bazı değerleri geçersiz kılmak ve bunlara özel mantık uygulamak için Sequence sınıfını kullanabilirsiniz.

```php
$users = User::factory()
                ->count(10)
                ->state(new Sequence(
                    ['admin' => 'Y'],
                    ['admin' => 'N'],
                ))
                ->create();
```

### Using factories with relationships

When using factories with relationships, Laravel also provides magic methods.

```php
// magic factory relationship methods
User::factory()->hasPosts(3)->create();

// instead of
User::factory()->has(Post::factory()->count(3))->create();
```

Tip given by [@oliverds\_](https://twitter.com/oliverds_/status/1441447356323430402)

### Herhangi bir olay göndermeden modeller oluşturun

Bazen herhangi bir olay göndermeden belirli bir modeli `update` isteyebilirsiniz. Bunu `updateQuietly` yöntemini kullanarak gerçekleştirebilirsiniz

```php
Post::factory()->createOneQuietly();

Post::factory()->count(3)->createQuietly();

Post::factory()->createManyQuietly([
    ['message' => 'A new comment'],
    ['message' => 'Another new comment'],
]);
```

### Useful for() method

Laravel factory çok kullanışlı bir `for()` metoduna sahiptir. Bunu `belongsTo()` ilişkileri oluşturmak için kullanabilirsiniz.

```php
public function run()
{
    Product::factory()
        ->count(3)
        ->for(Category::factory()->create())
        ->create();
}
```

Tip given by [@mmartin_joo](https://twitter.com/mmartin_joo/status/1461002439629361158)

### Run factories without dispatching events

Factory'yi kullanarak herhangi bir Event'i tetiklemeden birden fazla kayıt oluşturmak istiyorsanız, kodunuzu bir WithoutEvents kapanışının içine sarabilirsiniz.

```php
$posts = Post::withoutEvents(function () {
    return Post::factory()->count(50)->create();
});
```

Tip given by [@TheLaravelDev](https://twitter.com/TheLaravelDev/status/1510965402666676227)

### run() yönteminde bağımlılıkları belirtin

Bağımlılıkları seeder'ınızın `run()` yönteminde belirtebilirsiniz.

```php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        $user = User::factory()->create();

        $this->callWith(EventSeeder::class, [
            'user' => $user
        ]);
    }
}
```

```php
class EventSeeder extends Seeder
{
    public function run(User $user)
    {
        Event::factory()
            ->when($user, fn($f) => $f->for('user'))
            ->for(Program::factory())
            ->create();
    }
}
```

Tip given by [@justsanjit](https://twitter.com/justsanjit/status/1514428294418079746)

