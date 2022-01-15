# Whatsapp Notifications Channel for Laravel

[![Latest Version on Packagist][ico-version]][link-packagist]
[![Software License][ico-license]](LICENSE.md)
[![Total Downloads][ico-downloads]][link-packagist]

This package makes it easy to send Whatsapp notification using [Venom API](https://github.com/orkestral/venom) with Laravel.

## Contents

- [Installation](#installation)
  - [Setting up your Whatsapp Key](#setting-up-your-whatsapp-bot)
  - [Retrieving SESSION ID](#retrieving-chat-id)
  - [Using in Lumen](#using-in-lumen)
  - [Proxy or Bridge Support](#proxy-or-bridge-support)
- [Usage](#usage)
  - [Text Notification](#text-notification)
  - [Send a Poll](#send-a-poll)
  - [Attach a Contact](#attach-a-contact)
  - [Attach an Audio](#attach-an-audio)
  - [Attach a Photo](#attach-a-photo)
  - [Attach a Document](#attach-a-document)
  - [Attach a Location](#attach-a-location)
  - [Attach a Video](#attach-a-video)
  - [Attach a GIF File](#attach-a-gif-file)
  - [Routing a Message](#routing-a-message)
  - [Handling Response](#handling-response)
  - [On-Demand Notifications](#on-demand-notifications)
  - [Sending to Multiple Recipients](#sending-to-multiple-recipients)
- [Available Methods](#available-methods)
  - [Shared Methods](#shared-methods)
  - [Whatsapp Message methods](#whatsapp-message-methods)
  - [Whatsapp Location methods](#whatsapp-location-methods)
  - [Whatsapp File methods](#whatsapp-file-methods)
  - [Whatsapp Contact methods](#whatsapp-contact-methods)
  - [Whatsapp Poll methods](#whatsapp-poll-methods)
- [Alternatives](#alternatives)
- [Changelog](#changelog)
- [Testing](#testing)
- [Security](#security)
- [Contributing](#contributing)
- [Credits](#credits)
- [License](#license)

## Installation

You can install the package via composer:

```bash
composer require felipedamacenoteodoro/whatsapp-notification-channel
```

## Setting up your Whatsapp Bot

Set your venom session [Venom Session](https://orkestral.github.io/venom/pages/Getting%20Started/creating-client.html) and configure your Whatsapp Session:

```php
# config/services.php

'whatsapp-bot-api' => [
    'session' => env('WHATSAPP_API_SESSION', 'YOUR API WHATSAPP SESSION HERE')
],
```

## Using in Lumen

If you're using this notification channel in your Lumen project, you will have to add the below code in your `bootstrap/app.php` file.

```php
# bootstrap/app.php

// Make sure to create a "config/services.php" file and add the config from the above step.
$app->configure('services');

# Register the notification service providers.
$app->register(Illuminate\Notifications\NotificationServiceProvider::class);
$app->register(NotificationChannels\Whatsapp\WhatsappServiceProvider::class);
```

## Proxy or Bridge Support

You may not be able to send notifications if Whatsapp API is not accessible in your country,
you can either set a proxy by following the instructions [here](http://docs.guzzlephp.org/en/stable/quickstart.html#environment-variables) or
use a web bridge by setting the `base_uri` config above with the bridge uri.

You can set `HTTPS_PROXY` in your `.env` file.

## Usage

You can now use the channel in your `via()` method inside the Notification class.

### Text Notification

```php
use NotificationChannels\Whatsapp\WhatsappMessage;
use Illuminate\Notifications\Notification;

class InvoicePaid extends Notification
{
    public function via($notifiable)
    {
        return ["whatsapp"];
    }

    public function toWhatsapp($notifiable)
    {
        $url = url('/invoice/' . $this->invoice->id);

        return WhatsappMessage::create()
            // Optional recipient user id.
            ->to($notifiable->whatsapp_user_id)
            // Markdown supported.
            ->content("Hello there!\nYour invoice has been *PAID*")

            // (Optional) Blade template for the content.
            // ->view('notification', ['url' => $url])

            // (Optional) Inline Buttons
            ->button('View Invoice', $url)
            ->button('Download Invoice', $url)
            // (Optional) Inline Button with callback. You can handle callback in your bot instance
            ->buttonWithCallback('Confirm', 'confirm_invoice ' . $this->invoice->id);
    }
}
```

Here's a screenshot preview of the above notification on Whatsapp Messenger:

![Laravel Whatsapp Notification Example](https://user-images.githubusercontent.com/1915268/66616627-39be6180-ebef-11e9-92cc-f2da81da047a.jpg)

### Send a List

```php
public function toWhatsapp($notifiable)
{
    return WhatsappList::create()
        ->to($notifiable)
        ->question("Aren't Laravel Notification Channels awesome?")
        ->choices(['Yes', 'YEs', 'YES']);
}
```

Preview:

![Laravel Whatsapp Poll Example](https://user-images.githubusercontent.com/60013703/143135248-1224a69b-3233-4686-8a59-d41517d8c722.png)

### Attach a Contact

```php
public function toWhatsapp($notifiable)
{
    return WhatsappContact::create()
            ->to($notifiable->whatsapp_user_id) // Optional
            ->firstName('Felipe')
            ->lastName('D. Teodoro') // Optional
            ->phoneNumber('00000000');
}
```

Preview:

![Laravel Whatsapp Contact Example](https://user-images.githubusercontent.com/60013703/143510191-1d0f8e08-bd9a-4be5-8978-e6561508b47a.png)

### Attach an Audio

```php
public function toWhatsapp($notifiable)
{
    return WhatsappFile::create()
            ->to($notifiable->whatsapp_user_id) // Optional
            ->content('Audio') // Optional Caption
            ->audio('/path/to/audio.mp3');
}
```

Preview:

![Laravel Whatsapp Audio Notification Example](https://user-images.githubusercontent.com/60013703/143334174-4d796910-185f-46e2-89ad-5ec7a1a438c9.png)

### Attach a Photo

```php
public function toWhatsapp($notifiable)
{
    return WhatsappFile::create()
        ->to($notifiable->whatsapp_user_id) // Optional
        ->content('Awesome *bold* text and [inline URL](http://www.example.com/)')
        ->file('/storage/archive/6029014.jpg', 'photo'); // local photo

        // OR using a helper method with or without a remote file.
        // ->photo('https://file-examples-com.github.io/uploads/2017/10/file_example_JPG_1MB.jpg');
}
```

Preview:

![Laravel Whatsapp Photo Notification Example](https://user-images.githubusercontent.com/1915268/66616792-daad1c80-ebef-11e9-9bdf-c0bc484cf037.jpg)

### Attach a Document

```php
public function toWhatsapp($notifiable)
{
    return WhatsappFile::create()
        ->to($notifiable->whatsapp_user_id) // Optional
        ->content('Did you know we can set a custom filename too?')
        ->document('https://file-examples-com.github.io/uploads/2017/10/file-sample_150kB.pdf', 'sample.pdf');
}
```

Preview:

![Laravel Whatsapp Document Notification Example](https://user-images.githubusercontent.com/1915268/66616850-10520580-ebf0-11e9-9122-4f4d263f3b53.jpg)

### Attach a Location

```php
public function toWhatsapp($notifiable)
{
    return WhatsappLocation::create()
        ->latitude('40.6892494')
        ->longitude('-74.0466891');
}
```

Preview:

![Laravel Whatsapp Location Notification Example](https://user-images.githubusercontent.com/1915268/66616918-54450a80-ebf0-11e9-86ea-d5264fe05ba9.jpg)

### Attach a Video

```php
public function toWhatsapp($notifiable)
{
    return WhatsappFile::create()
        ->content('Sample *video* notification!')
        ->video('https://file-examples-com.github.io/uploads/2017/04/file_example_MP4_480_1_5MG.mp4');
}
```

Preview:

![Laravel Whatsapp Video Notification Example](https://user-images.githubusercontent.com/1915268/66617038-ed742100-ebf0-11e9-865a-bf0245d2cbbb.jpg)

### Attach a GIF File

```php
public function toWhatsapp($notifiable)
{
    return WhatsappFile::create()
        ->content('Woot! We can send animated gif notifications too!')
        ->animation('https://sample-videos.com/gif/2.gif');

        // Or local file
        // ->animation('/path/to/some/animated.gif');
}
```

Preview:

![Laravel Whatsapp Gif Notification Example](https://user-images.githubusercontent.com/1915268/66617071-109ed080-ebf1-11e9-989b-b237f2b9502d.jpg)

### Routing a Message

You can either send the notification by providing with the chat ID of the recipient to the `to($chatId)` method like shown in the previous examples or add a `routeNotificationForWhatsapp()` method in your notifiable model:

```php
/**
 * Route notifications for the Whatsapp channel.
 *
 * @return int
 */
public function routeNotificationForWhatsapp()
{
    return $this->whatsapp_user_id;
}
```

### Handling Response

You can make use of the [notification events](https://laravel.com/docs/5.8/notifications#notification-events) to handle the response from Whatsapp. On success, your event listener will receive a [Message](https://core.whatsapp.org/bots/api#message) object with various fields as appropriate to the notification type.

For a complete list of response fields, please refer the Whatsapp Bot API's [Message object](https://core.whatsapp.org/bots/api#message) docs.

### On-Demand Notifications

> Sometimes you may need to send a notification to someone who is not stored as a "user" of your application. Using the `Notification::route` method, you may specify ad-hoc notification routing information before sending the notification. For more details, you can check out the [on-demand notifications][link-on-demand-notifications] docs.

```php
use Illuminate\Support\Facades\Notification;

Notification::route('whatsapp', 'Whatsapp_CHAT_ID')
            ->notify(new InvoicePaid($invoice));
```

### Sending to Multiple Recipients

Using the [notification facade][link-notification-facade] you can send a notification to multiple recipients at once.

> If you're sending bulk notifications to multiple users, the Whatsapp Bot API will not allow more than 30 messages per second or so. 
> Consider spreading out notifications over large intervals of 8—12 hours for best results.
>
> Also note that your bot will not be able to send more than 20 messages per minute to the same group. 
>
> If you go over the limit, you'll start getting `429` errors. For more details, refer Whatsapp Bots [FAQ](https://core.whatsapp.org/bots/faq#broadcasting-to-users).

```php
use Illuminate\Support\Facades\Notification;

// Recipients can be an array of chat IDs or collection of notifiable entities.
Notification::send($recipients, new InvoicePaid());
```

## Available Methods

### Shared Methods

> These methods are optional and shared across all the API methods.

- `to(int|string $chatId)`: Recipient's chat id.
- `token(string $token)`: Bot token if you wish to override the default token for a specific notification.
- `button(string $text, string $url, int $columns = 2)`: Adds an inline "Call to Action" button. You can add as many as you want, and they'll be placed 2 in a row by default.
- `buttonWithCallback(string $text, string $callback_data, int $columns = 2)`: Adds an inline button with the given callback data. You can add as many as you want, and they'll be placed 2 in a row by default.
- `disableNotification(bool $disableNotification = true)`: Send the message silently. Users will receive a notification with no sound.
- `options(array $options)`: Allows you to add additional params or override the payload.
- `getPayloadValue(string $key)`: Get payload value for given key.

### Whatsapp Message methods

For more information on supported parameters, check out these [docs](https://whatsapp-bot-sdk.readme.io/docs/sendmessage).

- `content(string $content, int $limit = null)`: Notification message, supports markdown. For more information on supported markdown styles, check out these [docs](https://whatsapp-bot-sdk.readme.io/reference#section-formatting-options).
- `view(string $view, array $data = [], array $mergeData = [])`: (optional) Blade template name with Whatsapp supported HTML or Markdown syntax content if you wish to use a view file instead of the `content()` method.
- `chunk(int $limit = 4096)`: (optional) Message chars chunk size to send in parts (For long messages). Note: Chunked messages will be rate limited to one message per second to comply with rate limitation requirements from Whatsapp.

### Whatsapp Location methods

- `latitude(float|string $latitude)`: Latitude of the location.
- `longitude(float|string $longitude)`: Longitude of the location.

### Whatsapp File methods

- `content(string $content)`: (optional) File caption, supports markdown. For more information on supported markdown styles, check out these [docs](https://whatsapp-bot-sdk.readme.io/reference#section-formatting-options).
- `view(string $view, array $data = [], array $mergeData = [])`: (optional) Blade template name with Whatsapp supported HTML or Markdown syntax content if you wish to use a view file instead of the `content()` method.
- `file(string|resource|StreamInterface $file, string $type, string $filename = null)`: Local file path or remote URL, `$type` of the file (Ex:`photo`, `audio`, `document`, `video`, `animation`, `voice`, `video_note`) and optionally filename with extension. Ex: `sample.pdf`. You can use helper methods instead of using this to make it easier to work with file attachment.
- `photo(string $file)`: Helper method to attach a photo.
- `audio(string $file)`: Helper method to attach an audio file (MP3 file).
- `document(string $file, string $filename = null)`: Helper method to attach a document or any file as document.
- `video(string $file)`: Helper method to attach a video file.
- `animation(string $file)`: Helper method to attach an animated gif file.
- `voice(string $file)`: Helper method to attach a voice note (`.ogg` file with OPUS encoded).
- `videoNote(string $file)`: Helper method to attach a video note file (Upto 1 min long, rounded square video).

### Whatsapp Contact methods

- `phoneNumber(string $phoneNumber)`: Contact phone number.
- `firstName(string $firstName)`: Contact first name.
- `lastName(string $lastName)`: (optional) Contact last name.
- `vCard(string $vCard)`: (optional) Contact vcard.

### Whatsapp Poll methods

- `question(string $question)`: Poll question.
- `choices(array $choices)`: Poll choices.

## Alternatives

For advance usage, please consider using [whatsapp-bot-sdk](https://github.com/irazasyed/whatsapp-bot-sdk) instead.

## Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information what has changed recently.

## Testing

```bash
$ composer test
```

## Security

If you discover any security related issues, please email syed@lukonet.com instead of using the issue tracker.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

## Credits

- [Irfaq Syed][link-author]
- [All Contributors][link-contributors]

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.

[ico-phpchat]: https://img.shields.io/badge/Slack-PHP%20Chat-5c6aaa.svg?style=flat-square&logo=slack&labelColor=4A154B
[ico-whatsapp]: https://img.shields.io/badge/@PHPChatCo-2CA5E0.svg?style=flat-square&logo=whatsapp&label=Whatsapp
[ico-version]: https://img.shields.io/packagist/v/laravel-notification-channels/whatsapp.svg?style=flat-square
[ico-license]: https://img.shields.io/badge/license-MIT-brightgreen.svg?style=flat-square
[ico-scrutinizer]: https://img.shields.io/scrutinizer/coverage/g/laravel-notification-channels/whatsapp.svg?style=flat-square
[ico-code-quality]: https://img.shields.io/scrutinizer/g/laravel-notification-channels/whatsapp.svg?style=flat-square
[ico-downloads]: https://img.shields.io/packagist/dt/laravel-notification-channels/whatsapp.svg?style=flat-square

[link-phpchat]: https://phpchat.co/?ref=laravel-channel-whatsapp
[link-whatsapp]: https://t.me/PHPChatCo
[link-repo]: https://github.com/laravel-notification-channels/whatsapp
[link-packagist]: https://packagist.org/packages/laravel-notification-channels/whatsapp
[link-scrutinizer]: https://scrutinizer-ci.com/g/laravel-notification-channels/whatsapp/code-structure
[link-code-quality]: https://scrutinizer-ci.com/g/laravel-notification-channels/whatsapp
[link-author]: https://github.com/irazasyed
[link-contributors]: ../../contributors
[link-notification-facade]: https://laravel.com/docs/8.x/notifications#using-the-notification-facade
[link-on-demand-notifications]: https://laravel.com/docs/8.x/notifications#on-demand-notifications
[link-whatsapp-docs-update]: https://core.whatsapp.org/bots/api#update
[link-whatsapp-docs-getupdates]: https://core.whatsapp.org/bots/api#getupdates
