# Как мне красиво распечатать JSON с помощью node.js?

Третий параметр `JSON.stringify` определяет вставку пробела для симпатичной печати. Это может быть строка или число (количество пробелов). Узел может писать в вашу файловую систему с помощью `fs`. Пример:

```javascript
var fs = require('fs');

fs.writeFile('test.json', JSON.stringify({ a:1, b:2, c:3 }, null, 4));
/* test.json:
{
     "a": 1,
     "b": 2,
     "c": 3,
}
*/

```

См. [JSON.stringify() документы в MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/stringify), [Node fs docs](https: //nodejs.org/api/fs.html#fs_fs_writefile_file_data_options_callback)

**********
[javascript](/tags/javascript.md)
[node.js](/tags/node.js.md)
