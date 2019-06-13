# получить последний элемент в объекте JavaScript

Да, есть способ с использованием `Object.keys (obj)`. Это объясняется в [этой странице](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/Object/keys):

```
var fruitObject = { 'a' : 'apple', 'b' : 'banana', 'c' : 'carrot' };
Object.keys(fruitObject); // this returns all properties in an array ["a", "b", "c"]
```

Если вы хотите получить значение последнего объекта, вы можете сделать это:

```
fruitObject[Object.keys(fruitObject).length-1] // "carrot"
```

Решение с использованием синтаксиса назначения деструктурирования ES6:

```
var temp = { 'a' : 'apple', 'b' : 'banana', 'c' : 'carrot' };
var { [Object.keys(temp).pop()]: lastItem } = temp;
console.info(lastItem); //"carrot"
```