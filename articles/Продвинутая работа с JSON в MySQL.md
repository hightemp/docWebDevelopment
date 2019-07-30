# Продвинутая работа с JSON в MySQL

У MySQL нет возможности напрямую индексировать документы JSON, но есть альтернатива: генерируемые столбцы.

С момента введения поддержки типа данных JSON в MySQL 5.7.8 не хватает одной вещи: способности индексировать значения JSON. Для того, чтобы обойти это ограничение, можно использовать генерируемые столбцы. Эта возможность, представленная в MySQL 5.7.5, позволяет разработчикам создавать столбцы, содержащие информацию, полученную из других столбцов, предопределенных выражений или вычислений. Генерируя столбец из значений JSON, а затем индексируя его, можно практически индексировать поле с JSON.

Набор данных в формате JSON, используемый в данной статье, можно скачать [на Гитхабе](https://github.com/compose-ex/mysql-json-indexing-generated-columns) . Он содержит список игроков со следующими элементами: идентификатор игрока, его имя и игры, в которые он играл (Battlefield, Crazy Tennis и Puzzler).

```json
{
    "id":1,
    "name":"Sally",
    "games_played":{
        "Battlefield":{
            "weapon":"sniper rifle",
            "rank":"Sergeant V",
            "level":20
        },
        "Crazy Tennis":{
            "won":4,
            "lost":1
        },
        "Puzzler":{
            "time":7
        }
    }
},
…
```

  

Поле Battlefield содержит любимое оружие игрока, его текущий ранг и уровень этого ранга. Crazy Tennis включает в себя количество выигранных и проигранных игр, а Puzzler содержит время, затраченное игроком на прохождение игры. Создадим начальную таблицу:

  

```SQL
CREATE TABLE `players` (  
    `id` INT UNSIGNED NOT NULL,
    `player_and_games` JSON NOT NULL,
    PRIMARY KEY (`id`)
);
```

  

Этот запрос создает таблицу `players` , состоящую из идентификатора и JSON-данных, а также устанавливает в поле `id` первичный ключ.

  

Нужно построить индекс по полю с JSON. Давайте посмотрим, что нужно добавить в команду `CREATE TABLE` .

  

## Генерация столбцов

  

Для создания генерируемых столбцов в операторе `CREATE TABLE` используется следующий синтаксис:

  

```SQL
`column_name` datatype GENERATED ALWAYS AS (expression)
```

  

Ключевыми словами здесь являются `GENERATED ALWAYS` и `AS` . Фраза `GENERATED ALWAYS` необязательна. Она необходима только в том случае, если вы хотите явно указать, что этот столбец таблицы — генерируемый. Необходимо, чтобы слово `AS` сопровождалось выражением, которое вернет значение для генерируемого столбца.

  

Начнем с этого:

  

```SQL
`names_virtual` VARCHAR(20) GENERATED ALWAYS AS ...
```

  

Cоздаем столбец с именем `names_virtual` длиной до 20 символов, в котором будем хранить значение поля «name» из объекта JSON. Обращаться к полю «name» в JSON будем с использованием MySQL-оператора `->>` , который эквивалентен написанию `JSON_UNQUOTE (JSON_EXTRACT (...))` . Эта конструкция вернет значение поля «name» из объекта JSON в качестве результата.

  

```SQL
`names_virtual` VARCHAR(20) GENERATED ALWAYS AS (`player_and_games` ->> '$.name')
```

  

Этот код означает, что мы берём поле c JSON `player_and_games` и извлекаем значение из JSON по ключу «name» — дочернее по отношению к корню.

  

Как и в большинстве определений столбцов, существует ряд ограничений и параметров, которые можно применить к столбцу.

  

```SQL
[VIRTUAL|STORED] [UNIQUE [KEY]] [[NOT] NULL] [[PRIMARY] KEY]
```

  

Уникальные для генерируемых столбцов ключевые слова `VIRTUAL` и `STORED` указывают на то, будут ли значения сохраняться в таблице.

  

Ключевое слово `VIRTUAL` используется по умолчанию. Оно означает, что значения столбца не сохраняются и не занимают место для хранения. Они вычисляются при каждом чтении строки. Если вы создаете индекс с виртуальным столбцом, значение всё же сохраняется — в индексе.

  

Ключевое слово `STORED` указывает, что значения вычисляются при записи данных в таблицу: при вставке или обновлении. В этом случае индексу не нужно сохранять значение.

  

Другие параметры — необязательные ограничения, которые гарантируют, что значения поля будут `NULL` или `NOT NULL` , а также добавления ограничений на индекс, например, `UNIQUE` или `PRIMARY KEY` . Для гарантии существования значения следует использовать `NOT NULL` при создании столбца, однако ограничения зависят от варианта использования. В примере будет использоваться `NOT NULL` , так как у игроков обязательно есть имя.

  

Запрос, создающий таблицу:

  

```SQL
CREATE TABLE `players` (  
   `id` INT UNSIGNED NOT NULL,
   `player_and_games` JSON NOT NULL,
   `names_virtual` VARCHAR(20) GENERATED ALWAYS AS (`player_and_games` ->> '$.name') NOT NULL, 
   PRIMARY KEY (`id`)
);
```

  

Заполнение таблицы тестовыми данными:

  

```SQL
INSERT INTO `players` (`id`, `player_and_games`) VALUES (1, '{  
    "id": 1,  
    "name": "Sally",
    "games_played":{    
       "Battlefield": {
          "weapon": "sniper rifle",
          "rank": "Sergeant V",
          "level": 20
        },                                                                                                                          
       "Crazy Tennis": {
          "won": 4,
          "lost": 1
        },  
       "Puzzler": {
          "time": 7
        }
      }
   }'
);
...
```

  

Содержимое таблицы `players` на [Гисте](https://gist.github.com/win0err/a4f75528217e6547468551d0f942aae5#file-1-sql) или…

  
<details><summary>С поехавшим форматированием</summary>



</details>
  

Таблица включает столбец `names_virtual` , в который вставлены все имена игроков. Структура таблицы `players` :

  

```SQL
SHOW COLUMNS FROM `players`;

+------------------+------------------+------+-----+---------+-------------------+
| Field            | Type             | Null | Key | Default | Extra             |
+------------------+------------------+------+-----+---------+-------------------+
| id               | int(10) unsigned | NO   | PRI | NULL    |                   |
| player_and_games | json             | NO   |     | NULL    |                   |
| names_virtual    | varchar(20)      | NO   |     | NULL    | VIRTUAL GENERATED |
+------------------+------------------+------+-----+---------+-------------------+
```

  

Поскольку мы не указали, является ли генерируемый столбец `VIRTUAL` или `STORED` , по умолчанию MySQL автоматически сделал столбец `VIRTUAL` . Чтобы проверить, являются ли столбцы `VIRTUAL` или `STORED` , просто запустите вышеуказанный запрос `SHOW COLUMNS` , и он покажет либо `VIRTUAL GENERATED` , либо `STORED GENERATED` .

  

Теперь, когда мы настроили таблицу и виртуальный столбец, добавим еще четыре столбца, используя операции `ALTER TABLE` и `ADD COLUMN` . Они будут содержать уровни Battlefield, выигранные и проигранные игры в теннис и время в Puzzler.

  

```SQL
ALTER TABLE `players` ADD COLUMN `battlefield_level_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played.Battlefield.level') NOT NULL AFTER `names_virtual`;  
ALTER TABLE `players` ADD COLUMN `tennis_won_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played."Crazy Tennis".won') NOT NULL AFTER `battlefield_level_virtual`;  
ALTER TABLE `players` ADD COLUMN `tennis_lost_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played."Crazy Tennis".lost') NOT NULL AFTER `tennis_won_virtual`;  
ALTER TABLE `players` ADD COLUMN `times_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played.Puzzler.time') NOT NULL AFTER `tennis_lost_virtual`;  
```

  

Опять же, запустив запрос `SHOW COLUMNS FROM players;` , мы видим, что рядом с ними все столбцы указаны как `VIRTUAL GENERATED` . Это означает, что мы успешно настроили новые созданные `VIRTUAL` столбцы.

  

Код [Гисте](https://gist.github.com/win0err/a4f75528217e6547468551d0f942aae5#file-2-sql) или…

  
<details><summary>С поехавшим форматированием</summary>



</details>
  

Выполнение запроса `SELECT` показывает нам все значения из `VIRTUAL COLUMNS` , которые должны выглядеть так:

  

Код [Гисте](https://gist.github.com/win0err/a4f75528217e6547468551d0f942aae5#file-3-sql) или…

  
<details><summary>С поехавшим форматированием</summary>



</details>
  

После добавления данные и создания генерируемых столбцов, мы можем создавать индекс для каждого из них, чтобы оптимизировать поиск…

  

## Индексирование генерируемых столбцов

  

При установке вторичных индексов на значения генерируемых столбцов `VIRTUAL` значения сохраняются в индексе. Это дает преимущества: размер таблицы не увеличивается, появляется возможность использования индексов в MySQL.

  

Давайте сделаем простой запрос к генерируемому столбцу, чтобы увидеть, как он выглядит, прежде чем индексировать его. Изучив детали запроса при выборе `names_virtual` и имени «Sally», получим следующее:

  

```SQL
EXPLAIN SELECT * FROM `players` WHERE `names_virtual` = "Sally"\G  
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: players
   partitions: NULL
         type: ALL
possible_keys: NULL  
          key: NULL
      key_len: NULL
          ref: NULL
         rows: 6
     filtered: 16.67
        Extra: Using where
```

  

Для этого запроса MySQL просматривает каждую строку, чтобы найти «Sally». Однако, можно получить совершенно другой результат, добавив индекс к столбцу:

  

```SQL
CREATE INDEX `names_idx` ON `players`(`names_virtual`);  
```

  

Теперь, выполняя тот же запрос, получаем:

  

```SQL
EXPLAIN SELECT * FROM `players` WHERE `names_virtual` = "Sally"\G  
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: players
   partitions: NULL
         type: ref
possible_keys: names_idx  
          key: names_idx
      key_len: 22
          ref: const
         rows: 1
     filtered: 100.00
        Extra: NULL
```

  

Как видно, индекс в столбце ускорил запрос, просматривая только одну строку вместо шести, используя индекс `names_idx` . Давайте создадим индексы для остальных виртуальных столбцов, следуя тому же синтаксису, что и `names_idx` :

  

```SQL
CREATE INDEX `times_idx` ON `players`(`times_virtual`);  
CREATE INDEX `won_idx` ON `players`(`tennis_won_virtual`);  
CREATE INDEX `lost_idx` ON `players`(`tennis_lost_virtual`);  
CREATE INDEX `level_idx` ON `players`(`battlefield_level_virtual`);  
```

  

Можно проверить, были ли проиндексированы все наши столбцы, запустив:  
Код [Гисте](https://gist.github.com/win0err/a4f75528217e6547468551d0f942aae5#file-4-sql) или…

  
<details><summary>С поехавшим форматированием</summary>



</details>
  

Теперь, когда созданы несколько индексов в генерируемых столбцах, давайте усложним поиск. В этом примере выбираются идентификаторы, имена, выигранные теннисные игры, уровень Battlefield и время Puzzler для игроков, которые имеют уровень выше 50, а также выигравших 50 теннисных игр. Все результаты будут упорядочены по возрастанию в соответствии с временем в Puzzler. Команда SQL и результаты будут выглядеть так:

  

```SQL
SELECT `id`, `names_virtual`, `tennis_won_virtual`, `battlefield_level_virtual`, `times_virtual` FROM `players` WHERE (`battlefield_level_virtual` > 50 AND  `tennis_won_virtual` > 50) ORDER BY `times_virtual` ASC;

+----+---------------+--------------------+---------------------------+---------------+
| id | names_virtual | tennis_won_virtual | battlefield_level_virtual | times_virtual |
+----+---------------+--------------------+---------------------------+---------------+
|  5 | Phil          |                130 |                        98 |             7 |
|  6 | Henry         |                 68 |                        87 |            17 |
+----+---------------+--------------------+---------------------------+---------------+
```

  

Давайте посмотрим, как MySQL выполнял этот запрос:

  

```
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: players
   partitions: NULL
         type: range
possible_keys: won_idx,level_idx  
          key: won_idx
      key_len: 4
          ref: NULL
         rows: 2
     filtered: 66.67
        Extra: Using where; Using filesort
```

  

При использовании индексов `win_idx` и `level_idx` MySQL приходилось обращаться к двум столбцам, чтобы вернуть желаемый результат. Если запрос должен выполнить полный просмотр таблицы с миллионом записей, это займёт очень много времени. Однако, с помощью генерируемых столбцов и их индексированием, MySQL показал очень быстрый результат и удобный способ поиска элементов в JSON-данных.

  

Тем не менее остается один вопрос: для чего нужны `STORED` генерируемые столбцц? Как их использовать и как они работают?

  

## Хранение значений в генерируемых столбцах

  

Использование ключевого слова `STORED` при настройке генерируемого столбца обычно не предпочтительно, поскольку в основном значения в таблице сохраняются дважды: поле с JSON и в `STORED` столбце. Тем не менее, существует [три сценария](http://mysqlserverteam.com/indexing-json-documents-via-virtual-columns/) , когда в MySQL нужно использовать столбец `STORED` :

  

1.  индексирование первичных ключей,
2.  полнотекстовый индекс/индекс R-tree,
3.  столбец, который часто выбирается.

  

Синтаксис добавления генерируемого `STORED` столбца, совпадает с созданием генерируемых столбцов `VIRTUAL` , за исключением того, что нужно добавить ключевое слово `STORED` :

  

```SQL
`id` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.id') STORED NOT NULL,
```

  

Чтобы посмотреть как использовать `STORED` , создадим еще одну таблицу. Она будет брать `id` из данных JSON и хранить его в `STORED` столбце. Установим `PRIMARY KEY` для столбца `id` :

  

```SQL
CREATE TABLE `players_two` (  
    `id` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.id') STORED NOT NULL,
    `player_and_games` JSON NOT NULL,
    `names_virtual` VARCHAR(20) GENERATED ALWAYS AS (`player_and_games` ->> '$.name') NOT NULL,
    `times_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played.Puzzler.time') NOT NULL,
    `tennis_won_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played."Crazy Tennis".won') NOT NULL,
    `tennis_lost_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played."Crazy Tennis".lost') NOT NULL,
    `battlefield_level_virtual` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.games_played.Battlefield.level') NOT NULL, 
    PRIMARY KEY (`id`),
    INDEX `times_index` (`times_virtual`),
    INDEX `names_index` (`names_virtual`),
    INDEX `won_index` (`tennis_won_virtual`),
    INDEX `lost_index` (`tennis_lost_virtual`),
    INDEX `level_index` (`battlefield_level_virtual`)
);
```

  

Добавим тот же набор данных в `player_two` , за исключением того, что удалим `id` , который ранее добавили в операцию `INSERT` :

  

```SQL
INSERT INTO `players_two` (`player_and_games`) VALUES ('{  
    "id": 1,  
    "name": "Sally",  
    "games_played":{    
...
);
```

  

После того, как данные были вставлены в таблицу, запустим `SHOW COLUMNS` в новой таблице, чтобы узнать, как MySQL создал столбцы. Обратите внимание, что поле `id` теперь — `STORED GENERATED` и содержит индекс `PRIMARY KEY` .

  

```SQL
SHOW COLUMNS FROM `players_two`;

+---------------------------+-------------+------+-----+---------+-------------------+
| Field                     | Type        | Null | Key | Default | Extra             |
+---------------------------+-------------+------+-----+---------+-------------------+
| id                        | int(11)     | NO   | PRI | NULL    | STORED GENERATED  |
| player_and_games          | json        | NO   |     | NULL    |                   |
| names_virtual             | varchar(20) | NO   | MUL | NULL    | VIRTUAL GENERATED |
| times_virtual             | int(11)     | NO   | MUL | NULL    | VIRTUAL GENERATED |
| tennis_won_virtual        | int(11)     | NO   | MUL | NULL    | VIRTUAL GENERATED |
| tennis_lost_virtual       | int(11)     | NO   | MUL | NULL    | VIRTUAL GENERATED |
| battlefield_level_virtual | int(11)     | NO   | MUL | NULL    | VIRTUAL GENERATED |
+---------------------------+-------------+------+-----+---------+-------------------+
```

  

Замечание об использовании `PRIMARY KEY` с генерируемыми столбцами: MySQL не позволит создавать первичные ключи для генерируемых `VIRTUAL` столбцов. На самом деле, если не указать `STORED` в поле `id` , MySQL выдает следующую ошибку:

  

```
ERROR 3106 (HY000): 'Defining a virtual generated column as primary key' is not supported for generated columns.  
```

  

В то же время, если не устанавливать индекс первичного ключа и попытаться вставить данные, MySQL выдает сообщение об ошибке:

  

```
ERROR 3098 (HY000): The table does not comply with the requirements by an external plugin.  
```

  

Это означает, что у таблицы нет первичного ключа. Поэтому нужно вернуться и пересоздать таблицу, либо удалить столбец `id` и добавить генерируемый `STORED` столбец с первичным ключом, например:

  

```SQL
ALTER TABLE `players_two` ADD COLUMN `id` INT GENERATED ALWAYS AS (`player_and_games` ->> '$.id') STORED PRIMARY KEY;  
```

  

## Вывод

  

В статье показано как эффективно хранить данные JSON в MySQL, а так же как создавать индексы благодаря генерируемым столбцам. Использование генерируемых столбцов позволит размещать индексы по определенным элементам данных JSON. Именно эта гибкость делает MySQL очень привлекательной для использования JSON.

**********
[MySQL](/tags/MySQL.md)
