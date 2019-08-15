# Отправка электронной почты с использованием SMTP-сервера Gmail из скрипта PHP

Если у нас есть веб-сайт, то нам, очевидно, нужно отправлять электронные письма пользователям. Это электронное письмо должно пройти через вашу страницу контактов с нами, через новостную рассылку, регистрацию пользователей и т. Д.

PHP предоставляет функцию `mail()`, которая используется для отправки электронной почты. Но есть ограничения при использовании метода mail (). Вы не можете отправлять электронную почту с локального сервера разработки. Другим недостатком является высокая вероятность того, что ваша электронная почта окажется в папке «Спам».

Чтобы избавиться от этих проблем, нам нужно использовать SMTP-сервер для отправки электронных писем.

В этой статье мы узнаем, как использовать [PHPMailer](https://github.com/PHPMailer/PHPMailer) и SMTP-сервер Gmail для отправки электронных писем.

### Установка

Сначала нам нужно установить библиотеку PHPMailer в нашем проекте. Рекомендуемый способ установки библиотеки - через [Composer](https://getcomposer.org/).

Откройте командную строку в корневом каталоге вашего проекта и выполните приведенную ниже команду.

```php
composer require phpmailer/phpmailer
```

Поскольку мы используем Gmail SMTP, нам нужно изменить некоторые настройки в нашей учетной записи Google. Войдите в свою учетную запись Google и нажмите Моя учетная запись. Перейдя на страницу «Моя учетная запись», нажмите «Безопасность». Прокрутите вниз до низа, и вы найдете настройки «Меньше безопасного доступа к приложению». Установите его на ВКЛ.

 ![less-secure-apps](/images/d89746888da2d9510b64a9f031eaecd5.gif)   ![less-secure-apps](/images/3349fb712a42891ae4c902a9d79fff2f.png)  

Далее нам нужно написать код с использованием библиотеки PHPMailer вместе с настройками SMTP-сервера Gmail.

### PHP-скрипт для отправки электронной почты с помощью Gmail SMTP-сервера

Откройте ваш файл PHP, где вам нужно написать код для электронной почты. Например, мы предполагаем, что у вас есть файл `sendemail.php` в корневом каталоге.

 **sendemail.php** 

```php
<?php
//Import PHPMailer classes into the global namespace
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

require_once 'vendor/autoload.php';

$mail = new PHPMailer(true);
?>
```

В приведенном выше коде мы включили среду библиотеки PHPMailer в наш файл. Далее, для отправки писем с использованием PHPMailer нам нужно передать адрес SMTP-сервера Gmail, SMTP-порт для аутентификации Gmail и SMTP (который является ничем иным, как вашим именем пользователя и паролем учетной записи Google).

```php
$mail->isSMTP();
$mail->Host = 'smtp.googlemail.com';  //gmail SMTP server
$mail->SMTPAuth = true;
$mail->Username = 'GMAIL_USERNAME';   //username
$mail->Password = 'GMAIL_PASSWORD';   //password
$mail->SMTPSecure = 'ssl';
$mail->Port = 465;                    //SMTP port
```

Мы настроили наши настройки SMTP-сервера Gmail. Теперь нам всем приятно отправлять электронные письма пользователю.

```php
$mail->setFrom('FROM_EMAIL_ADDRESS', 'FROM_NAME');
$mail->addAddress('RECEPIENT_EMAIL_ADDRESS', 'RECEPIENT_NAME');

$mail->isHTML(true);

$mail->Subject = 'Email subject';
$mail->Body    = '<b>Email Body</b>';

$mail->send();
echo 'Message has been sent';
```

### Отправка вложений по электронной почте

Библиотека PHPMailer позволяет отправлять одно или несколько вложений в письме. Все что нам нужно сделать - это передать путь каталога наших вложений в метод `addAttachment`.

```php
$mail->addAttachment(__DIR__ . '/attachment1.png');
$mail->addAttachment(__DIR__ . '/attachment2.jpg');
```

Наш окончательный код выглядит следующим образом.

 **sendemail.php** 

```php
<?php
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\Exception;

require_once "vendor/autoload.php";
require_once "constants.php";

$mail = new PHPMailer(true);

try {
    $mail->isSMTP();
    $mail->Host = 'smtp.googlemail.com';  //gmail SMTP server
    $mail->SMTPAuth = true;
    $mail->Username = GMAIL_USERNAME;   //username
    $mail->Password = GMAIL_PASSWORD;   //password
    $mail->SMTPSecure = 'ssl';
    $mail->Port = 465;                    //smtp port
 
    $mail->setFrom('FROM_EMAIL_ADDRESS', 'FROM_NAME');
    $mail->addAddress('RECEPIENT_EMAIL_ADDRESS', 'RECEPIENT_NAME');

    $mail->addAttachment(__DIR__ . '/attachment1.png');
    $mail->addAttachment(__DIR__ . '/attachment2.png');

    $mail->isHTML(true);
    $mail->Subject = 'Email Subject';
    $mail->Body    = '<b>Email Body</b>';

    $mail->send();
    echo 'Message has been sent';
} catch (Exception $e) {
    echo 'Message could not be sent. Mailer Error: '. $mail->ErrorInfo;
}
?>
```

Мы надеемся, что вы понимаете, отправлять электронную почту, используя SMTP-сервер Gmail из сценария PHP. Вам также может понравиться наша статья [Отправка электронной почты через SMTP-сервер Gmail в Laravel](https://artisansweb.net/sending-email-via-gmail-smtp-server-laravel). Если у вас есть какие-либо вопросы или предложения, пожалуйста, оставьте комментарий ниже.

Если вам понравилась эта статья, подпишитесь на наш канал [Youtube](https://www.youtube.com/channel/UCosi8Kv8-EPLt5TBJLlsWJA?sub_confirmation=1) для видеоруководств.

**********
[PHP](/tags/PHP.md)
