# Объединить два объекта JavaScript

Расширьте объект JavaScript парами ключ / значение другого.

* * *

С помощью следующего помощника вы можете объединить два объекта в один новый объект:

```javscript
function extend(obj, src) {
    for (var key in src) {
        if (src.hasOwnProperty(key)) obj[key] = src[key];
    }
    return obj;
}

// example
var a = { foo: true }, b = { bar: false };
var c = extend(a, b);

console.log(c);
// { foo: true, bar: false }
```

Как правило, это полезно при объединении опций с настройками по умолчанию в функции или плагине.

Если поддержка IE 8 не требуется, вы можете использовать вместо этого `Object.keys` для той же функциональности:

```javascript
function extend(obj, src) {
    Object.keys(src).forEach(function(key) { obj[key] = src[key]; });
    return obj;
}
```


**********
[javascript](/tags/javascript.md)
