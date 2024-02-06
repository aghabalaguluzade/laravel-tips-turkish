## Mail

⬆️ [Go to main menu](README.md#laravel-tips) ⬅️ [Previous (Auth)](auth.md) ➡️ [Next (Artisan)](artisan.md)

- [E-postayı laravel.log'da test etme](#e-postayı-laravellogda-test-etme)
- [Laravel'de e-posta eki olarak kullanmak için dosyaları saklamak zorunda değilsiniz](#laravelde-e-posta-eki-olarak-kullanmak-için-dosyaları-saklamak-zorunda-değilsiniz)
- [Mailables Önizleme](#mailables-önizleme)
- [Mailables olmadan Posta Önizleme](#mailables-olmadan-posta-önizleme)
- [Laravel bildirimlerinde varsayılan e-posta konusu](#laravel-bildirimlerinde-varsayılan-e-posta-konusu)
- [Herkese bildirim gönder](#herkese-bildirim-gönder)
- [Koşullu nesne özelliklerini ayarlama](#koşullu-nesne-özelliklerini-ayarlama)

### E-postayı laravel.log'da test etme

Uygulamanızda e-posta içeriklerini test etmek istiyorsanız ancak Mailgun gibi bir şey kuramıyorsanız veya kurmak istemiyorsanız, `.env` parametresini `MAIL_DRIVER=log` kullanın ve tüm e-postalar gerçekten gönderilmek yerine `storage/logs/laravel.log` dosyasına kaydedilecektir.

### Laravel'de e-posta eki olarak kullanmak için dosyaları saklamak zorunda değilsiniz

Kullanıcı tarafından yüklenen dosyaları Mailables'a eklemek için **attachData**'yı kullanmanız yeterlidir.

İşte bunu kullanan Mailable sınıfından bir örnek.
```php
public function build()
{
     return $this->subject('Inquiry')
          ->to('example@example.com')
          ->markdown('email.inquiry')
          ->attachData(
               $this->file,
               $this->file->getClientOriginalName(),
          );
}
```

Tip given by [@ecrmnn](https://twitter.com/ecrmnn/status/1570449885664808961)

### Mailables Önizleme

E-posta göndermek için Mailables'ı kullanıyorsanız, sonucu göndermeden doğrudan tarayıcınızda önizleyebilirsiniz. Rota sonucu olarak bir Mailable döndürmeniz yeterlidir:

```php
Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return new App\Mail\InvoicePaid($invoice);
});
```

### Mailables olmadan Posta Önizleme

E-postanızı Mailables olmadan da önizleyebilirsiniz. Örneğin, bildirim oluştururken posta bildiriminiz için kullanılabilecek işaretlemeyi belirtebilirsiniz.

```php
use Illuminate\Notifications\Messages\MailMessage;

Route::get('/mailable', function () {
    $invoice = App\Invoice::find(1);
    return (new MailMessage)->markdown('emails.invoice-paid', compact('invoice'));
});
```

Ayrıca `MailMessage` nesnesi tarafından sağlanan `view` ve diğerleri gibi diğer yöntemleri de kullanabilirsiniz.

Tip given by [@raditzfarhan](https://github.com/raditzfarhan)

### Laravel bildirimlerinde varsayılan e-posta konusu

Laravel Bildirimi gönderirseniz ve **toMail()** içinde konu belirtmezseniz, varsayılan konu bildirim sınıfınızın adıdır ve CamelCased ile boşluklara ayrılır.

Yani, eğer varsa:

```php
class UserRegistrationEmail extends Notification {
    //
}
```

Ardından **Kullanıcı Kayıt E-postası** konulu bir e-posta alacaksınız.

### Herkese bildirim gönder

Laravel Bildirimlerini sadece `$user->notify()` ile belirli bir kullanıcıya değil, aynı zamanda `Notification::route()` ile "isteğe bağlı" bildirimler olarak adlandırılan istediğiniz herkese gönderebilirsiniz:

```php
Notification::route('mail', 'taylor@example.com')
        ->route('nexmo', '5555555555')
        ->route('slack', 'https://hooks.slack.com/services/...')
        ->notify(new InvoicePaid($invoice));
```

### Koşullu nesne özelliklerini ayarlama

Eylem çağrısı (call to action) gibi koşullu nesne özelliklerini ayarlamak için MailMessage bildirimlerinizde `when()` veya `unless()` yöntemlerini kullanabilirsiniz.

```php
class InvoicePaid extends Notification
{
    public function toMail(User $user)
    {
        return (new MailMessage)
            ->success()
            ->line('We\'ve received your payment')
            ->when($user->isOnMonthlyPaymentPlan(), function (MailMessage $message) {
                $message->action('Save 20% by paying yearly', route('account.billing'));
            })
            ->line('Thank you for using Unlock.sh');
    }
}
```

"Illuminate\Support\Traits\Conditionable" trait'ini kullanarak kendi sınıflarınızda `when` veya `unless` yöntemlerini kullanın

Tip given by [@Philo01](https://twitter.com/Philo01/status/1503302749525528582)

