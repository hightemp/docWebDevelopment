# PHP + curl, HTTP POST пример кода?

## Процедурный

```
// установить поля сообщения
$post = [
    'username' => 'user1',
    'password' => 'passuser1',
    'gender'   => 1,
];

$ch = curl_init('http://www.example.com');
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_POSTFIELDS, $post);

// выполнить!
$response = curl_exec($ch);

// закрыть соединение, освободить используемые ресурсы
curl_close($ch);

// делайте что угодно с вашим ответом
var_dump($response);
```

## Объектно-ориентированный

```
<?php

// с учетом соответствующих изменений
namespace MyApp\Http;

class CurlPost
{
    private $url;
    private $options;

    /**
     * @param string $url     Request URL
     * @param array  $options cURL options
     */
    public function __construct($url, array $options = [])
    {
        $this->url = $url;
        $this->options = $options;
    }

    /**
     * Get the response
     * @return string
     * @throws \RuntimeException On cURL error
     */
    public function __invoke(array $post)
    {
        $ch = curl_init($this->url);

        foreach ($this->options as $key => $val) {
            curl_setopt($ch, $key, $val);
        }

        curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
        curl_setopt($ch, CURLOPT_POSTFIELDS, $post);

        $response = curl_exec($ch);
        $error    = curl_error($ch);
        $errno    = curl_errno($ch);

        if (is_resource($ch)) {
            curl_close($ch);
        }

        if (0 !== $errno) {
            throw new \RuntimeException($error, $errno);
        }

        return $response;
    }
}
```

### использование

```
// создать объект curl
$curl = new \MyApp\Http\CurlPost('http://www.example.com');

try {
    // выполнить запрос
    echo $curl([
        'username' => 'user1',
        'password' => 'passuser1',
        'gender'   => 1,
    ]);
} catch (\RuntimeException $ex) {
    // ловить ошибки
    die(sprintf('Http error %s with code %d', $ex->getMessage(), $ex->getCode()));
}
```

Обратите внимание: лучше создать интерфейс с именем AdapterInterface, например, с методом getResponse(), и позволить классу выше реализовать его. Затем вы всегда можете поменять эту реализацию с другим адаптером по вашему вкусу, без каких-либо побочных эффектов для вашего приложения.

## Использование HTTPS / шифрование трафика

Обычно есть проблема с cURL в PHP под операционной системой Windows. При попытке подключиться к защищенной https конечной точке вы получите сообщение об ошибке «сбой проверки сертификата».

Большинство людей делают здесь, чтобы заставить библиотеку cURL просто игнорировать ошибки сертификата и продолжать (`curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false);`). Поскольку это заставит ваш код работать, вы создадите огромную дыру в безопасности и дадите злоумышленникам возможность выполнять различные атаки на ваше приложение, например [Человек посередине](https://wikipedia.org/wiki/Man-in-the-middle_attack) атака или что-то подобное.

Никогда, никогда не делай этого. Вместо этого вам просто нужно изменить ваш php.ini и сообщить PHP, где находится ваш файл CA Certificate, чтобы он мог правильно проверять сертификаты:

```
; изменить абсолютный путь к файлу cacert.pem
curl.cainfo=c:\php\cacert.pem
```

Последний `cacert.pem` может быть загружен из Интернета или [извлечен из вашего любимого браузера](https://www.google.pl/search?q=how%20extract%20cacert.pem). При изменении любых настроек, связанных с `php.ini`, не забудьте перезапустить ваш веб-сервер.

**********
[PHP](/tags/PHP.md)
[CURL](/tags/CURL.md)
