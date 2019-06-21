# Делаем GraphQL API на PHP и MySQL. Часть 1: Установка, схема и запросы

 В последнее время я все чаще и чаще слышу про GraphQL. И в интернете уже можно найти немало статей о том как сделать свой GraphQL сервер. Но почти во всех этих статьях в качестве бэкенда используется Node.js.   
  
 Я ничего не имею против Node.js и сам с удовольствием использую его, но все-таки большую часть проектов я делаю на PHP. К тому же хостинг с PHP и MySQL гораздо дешевле и доступнее чем хостинг с Node.js. Поэтому мне кажется не справедливым тот факт, что об использовании GraphQL на PHP в интернете практически нет ни слова.   
  
 В данной статье я хочу рассказать о том, как сделать свой GraphQL сервер на PHP с помощью библиотеки   [graphql-php](https://github.com/webonyx/graphql-php)   и как с его помощью реализовать простое API для получения данных из MySQL.   
  
 Я решил отказаться от использования какого-либо конкретного PHP фреймворка в данной статье, но после ее прочтения вам не составит особого труда применить эти знания в своем приложении. Тем более для некоторых фреймворков уже есть свои   [библиотеки](https://github.com/chentsulin/awesome-graphql#lib-php)   основанные на graphql-php, которые облегчат вашу работу.   
  

## Подготовка

  
 В данной статье я не буду делать фронтенд, поэтому для удобного тестирования запросов к GraphQL серверу рекомендую установить GraphiQL-расширение для браузера.   
  
 Для Chrome это могут быть:   
  

*    [ChromeiQL](https://chrome.google.com/webstore/detail/chromeiql/fkkiamalmpiidkljmicmjfbieiclmeij) 
*    [GraphiQL Feen](https://chrome.google.com/webstore/detail/graphiql-feen/mcbfdonlkfpbfdpimkjilhdneikhfklp) 

  
 Также понадобится создать таблицы в БД и заполнить их тестовым набором данных.   
  
 В таблице «users» будем хранить список пользователей:   
  
 [ ![Таблица users](/images/86892ab6b448fdf4099d910979ecc84e.png) ](https://habrastorage.org/web/532/d35/ff3/532d35ff3a0d4275956d450bac813d64.png)   
  
 А в таблице «friendships» связи типа «многие-ко-многим», которые будут обозначать дружбу между пользователями:   
  
 [ ![Таблица friendships](/images/99c7dfa8d0ee50a25508f4224b8c0a58.png) ](https://habrastorage.org/web/c64/227/a03/c64227a036ef47bebc6e4eaab5ff4706.png)   
  
 Дамп базы данных, как и весь остальной код, можно взять из   [репозитория данной статьи на Github](https://github.com/XAHTEP26/graphql-php-tutorial/tree/master/get-data-from-mysql)  .   
  

## Hello, GraphQL!

  
 Для начала необходимо установить   [graphql-php](https://github.com/webonyx/graphql-php)   в наш проект. Можно сделать это с помощью composer:   
  

```
composer require webonyx/graphql-php
```

  
 Теперь, по традиции напишем «Hello, World».   
  
 Для этого в корне создадим файл graphql.php, который будет служить конечной точкой (endpoint) нашего GraphQL сервера.   
  
 В нем подключим автозагрузчик composer:   
  

```
require_once __DIR__ . '/vendor/autoload.php';
```

  
 Подключим GraphQL:   
  

```
use GraphQL\GraphQL;
```

  
 Чтобы заставить GraphQL выполнить запрос необходимо передать ему сам запрос и схему данных.   
  
 Для получения запроса напишем следующий код:   
  

```
$rawInput = file_get_contents('php://input');
$input = json_decode($rawInput, true);
$query = $input['query'];
```

  
 Чтобы создать схему сначала подключим GraphQL\Schema:   
  

```
use GraphQL\Schema;
```

  
 Конструктор схемы принимает массив, в котором должен быть указан корневой тип данных Query, который служит для чтения данных вашего API, поэтому сначала создадим этот тип.   
  
<details><summary>Примечание</summary>

 _Также схема может содержать необязательный корневой тип данных Mutation, который предоставляет API для записи данных, но в рамках данной статьи мы его рассматривать не будем._   


</details>
  
 В простейшем случае тип данных Query должен быть экземпляром класса ObjectType, а его поля должны быть простых типов (например int или string), поэтому подключим классы предоставляющие эти типы данных в GraphQL:   
  

```
use GraphQL\Type\Definition\Type;
use GraphQL\Type\Definition\ObjectType;
```

  
 И создадим тип данных Query:   
  

```
$queryType = new ObjectType([
    'name' => 'Query',
    'fields' => [
        'hello' => [
            'type' => Type::string(),
            'description' => 'Возвращает приветствие',
            'resolve' => function () {
                return 'Привет, GraphQL!';
            }
        ]
    ]
]);
```

  
 Как можно заметить тип данных обязательно должен содержать имя (name) и массив полей (fields), а также можно указать необязательное описание (description).   
  
<details><summary>Примечание</summary>

 _Также тип данных может содержать свойства «interfaces», «isTypeOf» и «resolveField», но в рамках данной статьи мы их рассматривать не будем._   


</details>
  
 Поля типа данных также должны иметь обязательные свойства «name» и «type». Если свойство «name» не задано, то в качестве имени используется ключ поля (в данном случае «hello»). Также в нашем примере у поля «hello» заданы необязательные свойства «description» — описание и «resolve» — функция возвращающая результат. В этом случае функция «resolve» просто возвращает строку "  _Привет, GraphQL!_  ", но в более сложной ситуации она может получать какую-либо информацию по API или из БД и обрабатывать ее.   
  
<details><summary>Примечание</summary>

 _Также поля могут содержать свойство args, которое будет рассмотрено в статье позже, и свойство deprecationReason, которое в данной статье не рассматривается._   


</details>
  
 Таким образом, мы создали корневой тип данных «Query», который содержит всего одно поле «hello», возвращающее простую строку текста. Давайте добавим его в схему данных:   
  

```
$schema = new Schema([
    'query' => $queryType
]);
```

  
 А затем выполним запрос GraphQL для получения результата:   
  

```
$result = GraphQL::execute($schema, $query);
```

  
 Остается только вывести результат в виде JSON и наше приложение готово:   
  

```
header('Content-Type: application/json; charset=UTF-8');
echo json_encode($result);
```

  
 Обернем код в блок try-catch, для обработки ошибок и в итоге код файла graphql.php будет выглядеть так:   
  
<details><summary>graphql.php</summary>



```
<?php

require_once __DIR__ . '/vendor/autoload.php';

use GraphQL\GraphQL;
use GraphQL\Schema;
use GraphQL\Type\Definition\Type;
use GraphQL\Type\Definition\ObjectType;

try {
    // Получение запроса
    $rawInput = file_get_contents('php://input');
    $input = json_decode($rawInput, true);
    $query = $input['query'];

    // Содание типа данных "Запрос"
    $queryType = new ObjectType([
        'name' => 'Query',
        'fields' => [
            'hello' => [
                'type' => Type::string(),
                'description' => 'Возвращает приветствие',
                'resolve' => function () {
                    return 'Привет, GraphQL!';
                }
            ]
        ]
    ]);

    // Создание схемы
    $schema = new Schema([
        'query' => $queryType
    ]);

    // Выполнение запроса
    $result = GraphQL::execute($schema, $query);
} catch (\Exception $e) {
    $result = [
        'error' => [
            'message' => $e->getMessage()
        ]
    ];
}

// Вывод результата
header('Content-Type: application/json; charset=UTF-8');
echo json_encode($result);
```

  


</details>
  
 Проверим работу GraphQL. Для этого запустим расширение для GraphiQL, установим endpoint (в моем случае это «  [localhost/graphql.php](http://localhost/graphql.php)  ») и выполним запрос:   
  
 [ ![Проверка работы GraphQL](/images/254763928906ca704dc8268c6e73da87.png) ](https://habrastorage.org/web/77e/f44/80b/77ef4480bf534f05bf88d482b5310ab6.png)   
  

## Вывод пользователей из БД

  
 Теперь усложним задачу. Выведем список пользователей из базы данных MySQL.   
  
 Для этого нам понадобится создать еще один тип данных класса ObjectType. Чтобы не нагромождать код в graphql.php, вынесем все типы данных в отдельные файлы. А чтобы у нас была возможность использовать типы данных внутри самих себя, оформим их в виде классов. Например, чтобы в типе данных «user» можно было добавить поле «friends», которое будет являться массивом пользователей такого же типа «user».   
  
 Когда мы оформляем тип данных в виде класса, то не обязательно указывать у него свойство «name», потому что оно по умолчанию берется из названия класса (например у класса QueryType будет имя Query).   
  
 Теперь корневой тип данных Query, который был в graphql.php:   
  

```
$queryType = new ObjectType([
    'name' => 'Query',
    'fields' => [
        'hello' => [
            'type' => Type::string(),
            'description' => 'Возвращает приветствие',
            'resolve' => function () {
                return 'Привет, GraphQL!';
            }
        ]
    ]
]);
```

  
 Будет находиться в отдельном файле QueryType.php и выглядеть так:   
  

```
class QueryType extends ObjectType
{
    public function __construct()
    {
        $config = [
            'fields' => [
                'hello' => [
                    'type' => Type::string(),
                    'description' => 'Возвращает приветствие',
                    'resolve' => function () {
                        return 'Привет, GraphQL!';
                    }
                ]
            ]
        ];
        parent::__construct($config);
    }
}
```

  
 А чтобы в дальнейшем избежать бесконечной рекурсии при определении типов, в свойстве «fields» лучше всегда указывать не массив полей, а анонимную функцию, возвращающую массив полей:   
  

```
class QueryType extends ObjectType
{
    public function __construct()
    {
        $config = [
            'fields' =>  function() {
                return [
                    'hello' => [
                        'type' => Type::string(),
                        'description' => 'Возвращает приветствие',
                        'resolve' => function () {
                            return 'Привет, GraphQL!';
                        }
                    ]
                ];
            }
        ];
        parent::__construct($config);
    }
}
```

  
 При разработке проекта может появиться очень много типов данных, поэтому для них лучше создать отдельный реестр, который будет служить фабрикой для всех типов данных, в том числе и базовых, используемых в проекте. Давайте создадим папку App, а в ней файл, Types.php, который как раз и будет тем самым реестром для всех типов данных проекта.   
  
 Также в папке App создадим подпапку Type, в которой будем хранить все наши типы данных и перенесем в нее QueryType.php.   
  
 Теперь добавим пространство имен и заполним реестр Types.php необходимыми типами:   
  
<details><summary>App/Types.php</summary>



```
<?php

namespace App;

use App\Type\QueryType;
use GraphQL\Type\Definition\Type;

class Types
{
    private static $query;

    public static function query()
    {
        return self::$query ?: (self::$query = new QueryType());
    }

    public static function string()
    {
        return Type::string();
    }
}
```

  


</details>
  
 Пока в нашем реестре будет всего 2 типа данных: 1 простой (string) и 1 составной (query).   
  
 Теперь во всех остальных файлах вместо:   
  

```
use GraphQL\Type\Definition\Type;
```

  
 Подключим наш реестр типов:   
  

```
use App\Types;
```

  
 И заменим все ранее указанные типы, на типы из реестра.   
  
 В QueryType.php вместо:   
  

```
Type::string()
```

  
 Будет:   
  

```
Types::string()
```

  
 А схема в graphql.php теперь будет выглядеть так:   
  

```
$schema = new Schema([
    'query' => Types::query()
]);
```

  
 Чтобы получить пользователей из базы данных, необходимо обеспечить интерфейс доступа к ней. Получать данные из базы можно любым способом. В каждом фреймворке для этого есть свои инструменты. Для данной статьи я написал простейший интерфейс который может подключаться к MySQL базе данных и выполнять в ней запросы. Так как это не относится к GraphQL, то я не буду объяснять как реализованы методы в данном классе, а просто приведу его код:   
  
<details><summary>App/DB.php</summary>



```
<?php

namespace App;

class DB
{
    private static $pdo;
    
    public static function init($config)
    {
        self::$pdo = new PDO("mysql:host={$config['host']};dbname={$config['database']}", $config['username'], $config['password']);
        self::$pdo->setAttribute(PDO::ATTR_DEFAULT_FETCH_MODE, PDO::FETCH_OBJ);
    }
    
    public static function selectOne($query)
    {
        $records = self::select($query);
        return array_shift($records);
    }
    
    public static function select($query)
    {
        $statement = self::$pdo->query($query);
        return $statement->fetchAll();
    }
    
    public static function affectingStatement($query)
    {
        $statement = self::$pdo->query($query);
        return $statement->rowCount();
    }
}
```

  


</details>
  
 В файле graphql.php добавим код для инициализации подключения к БД:   
  

```
// Настройки подключения к БД
$config = [
    'host' => 'localhost',
    'database' => 'gql',
    'username' => 'root',
    'password' => 'root'
];

// Инициализация соединения с БД
DB::init($config);
```

  
 Теперь в папке Type создадим тип данных User, который будет отображать данные о пользователе. Код файла UserType.php будет таким:   
  
<details><summary>App/Type/UserType.php</summary>



```
<?php

namespace App\Type;

use App\DB;
use App\Types;
use GraphQL\Type\Definition\ObjectType;

class UserType extends ObjectType
{
    public function __construct()
    {
        $config = [
            'description' => 'Пользователь',
            'fields' => function() {
                return [
                    'id' => [
                        'type' => Types::string(),
                        'description' => 'Идентификатор пользователя'
                    ],
                    'name' => [
                        'type' => Types::string(),
                        'description' => 'Имя пользователя'
                    ],
                    'email' => [
                        'type' => Types::string(),
                        'description' => 'E-mail пользователя'
                    ],
                    'friends' => [
                        'type' => Types::listOf(Types::user()),
                        'description' => 'Друзья пользователя',
                        'resolve' => function ($root) {
                            return DB::select("SELECT u.* from friendships f JOIN users u ON u.id = f.friend_id WHERE f.user_id = {$root->id}");
                        }
                    ],
                    'countFriends' => [
                        'type' => Types::int(),
                        'description' => 'Количество друзей пользователя',
                        'resolve' => function ($root) {
                            return DB::affectingStatement("SELECT u.* from friendships f JOIN users u ON u.id = f.friend_id WHERE f.user_id = {$root->id}");
                        }
                    ]
                ];
            }
        ];
        parent::__construct($config);
    }
}
```

  


</details>
  
 Значение полей можно понять из их свойства «description». Свойства «id», «name», «email» и «countFriends» имеют простые типы, а свойство «friends» является списком друзей – таких же пользователей, поэтому имеет тип:   
  

```
Types::listOf(Types::user())
```

  
 Необходимо также добавить в наш реестр пару базовых типов, которые мы раньше не использовали:   
  

```
public static function int()
{
    return Type::int();
}

public static function listOf($type)
{
    return Type::listOf($type);
}
```

  
 И только, что созданный нами тип User:   
  

```
private static $user;

public static function user()
{
    return self::$user ?: (self::$user = new UserType());
}
```

  
 Возвращаемые значения (resolve) для свойств «friends» и «countFriends» берутся из базы данных. Анонимная функция в «resolve» первым аргументом получает значение текущего поля ($root), из которого можно узнать id пользователя для вставки его в запрос списка друзей.   
  
<details><summary>Примечание</summary>

 _Я не уделял внимание экранированию запросов, потому как этот код написан в учебных целях, но на реальном проекте так делать, конечно, нельзя._   


</details>
  
 В завершении изменим код QueryType.php так, чтобы в API были поля для получения информации о конкретном пользователе по его идентификатору (поле «user»), а также для получения списка всех пользователей (поле «allUsers»):   
  
<details><summary>App/Type/QueryType.php</summary>



```
<?php

namespace App\Type;

use App\DB;
use App\Types;
use GraphQL\Type\Definition\ObjectType;

class QueryType extends ObjectType
{
    public function __construct()
    {
        $config = [
            'fields' => function() {
                return [
                    'user' => [
                        'type' => Types::user(),
                        'description' => 'Возвращает пользователя по id',
                        'args' => [
                            'id' => Types::int()
                        ],
                        'resolve' => function ($root, $args) {
                            return DB::selectOne("SELECT * from users WHERE id = {$args['id']}");
                        }
                    ],
                    'allUsers' => [
                        'type' => Types::listOf(Types::user()),
                        'description' => 'Список пользователей',
                        'resolve' => function () {
                            return DB::select('SELECT * from users');
                        }
                    ]
                ];
            }
        ];
        parent::__construct($config);
    }
}
```

  


</details>
  
 Тут чтобы узнать идентификатор пользователя, данные которого необходимо получить, у поля «user» мы используем свойство «args», в котором содержится массив аргументов. Массив «args» передается в анонимную функцию «resolve» вторым аргументом, используя который мы узнаем id целевого пользователя.   
  
<details><summary>Примечание</summary>

 _У аргументов могут быть свои свойства, но в этом случае я использую упрощенную форму записи массива аргументов, при которой ключи массива являются именами, а значения – типами аргументов:_   

```
'args' => [
    'id' => Types::int()
]
```

  
 _Вместо:_   

```
'args' => [
    'id' => [
        'type' => Types::int()
    ]
]
```

  


</details>
  
 Теперь можно запустить сервер GraphQL и проверить его работу таким запросом:   
  
 [ ![Проверка работы GraphQL: Получение данных пользователя](/images/780544e2cc3a2276b73905f6af536769.png) ](https://habrastorage.org/web/421/d7e/8f0/421d7e8f01c246cf8ae69fa15dd02df5.png)   
  
 Или таким:   
  
 [ ![Проверка работы GraphQL: Получение данных пользователя](/images/ed3697559f9072b7ef1bd127cf4f0b67.png) ](https://habrastorage.org/web/5e0/0e5/144/5e00e51444334aa5bd5a869abc93fe89.png)   
  
 Или любым другим.   
  

## Заключение

  
 На этом все. Читайте   [документацию](http://webonyx.github.io/graphql-php/)  . Задавайте вопросы в комментариях. Критикуйте.   
  
 Также рекомендую почитать   [исходный код с комментариями на Github](https://github.com/XAHTEP26/graphql-php-tutorial/tree/master/get-data-from-mysql)  .   
  
 Другие части данной статьи:   

1.  Установка, схема и запросы
2.   [Мутации, переменные, валидация и безопасность](https://habrahabr.ru/post/329238/) 
3.   [Решение проблемы N+1 запросов](https://habrahabr.ru/post/329408/)

**********
[PHP](/tags/PHP.md)
[GraphQL](/tags/GraphQL.md)
[MySQL](/tags/MySQL.md)
