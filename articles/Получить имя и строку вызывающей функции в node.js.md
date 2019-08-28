# Получить имя и строку вызывающей функции в node.js

Используя информацию здесь: [Доступ к номеру строки в V8 JavaScript (Chrome & Node.js)](https://stackoverflow.com/questions/11386492/accessing-line-number-in-v8-javascript-chrome-node-js )

Вы можете добавить несколько прототипов, чтобы обеспечить доступ к этой информации из V8:

```javascript
Object.defineProperty(global, '__stack', {
get: function() {
        var orig = Error.prepareStackTrace;
        Error.prepareStackTrace = function(_, stack) {
            return stack;
        };
        var err = new Error;
        Error.captureStackTrace(err, arguments.callee);
        var stack = err.stack;
        Error.prepareStackTrace = orig;
        return stack;
    }
});

Object.defineProperty(global, '__line', {
get: function() {
        return __stack[1].getLineNumber();
    }
});

Object.defineProperty(global, '__function', {
get: function() {
        return __stack[1].getFunctionName();
    }
});

function foo() {
    console.log(__line);
    console.log(__function);
}

foo()

```

Возвращает '28' и 'foo' соответственно.

**********
[javascript](/tags/javascript.md)
