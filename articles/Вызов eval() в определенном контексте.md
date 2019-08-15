# Вызов eval() в определенном контексте

**Определенно** не правильный ответ, и, пожалуйста, не используйте инструкцию `with` , если вы не знаете, что делаете, но для любопытных вы можете сделать это

### Пример

```javascript
    var a = {b: "foo"};
    with(a) {
        // prints "foo"
        console.log(eval("b"));  
        
        // however, "this.b" prints undefined
        console.log(eval("this.b"));
    
        // because "this" is still the window for eval
        // console.log(eval("this")); // prints window

// if you want to fix, you need to wrap with a function, as the main answer pointed out
        (function(){
	         console.log(eval("this.b")); // prints foo
        }).call(a);     
    }
    
    // so if you want to support both    
    with (a) {
    	(function (){
        console.log("--fix--");
      	console.log(eval("b")); // foo
        console.log(eval("this.b")); // foo
      }).call(a);
    }
```

[ `with` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/with) \- это неудачная попытка создания блочных областей внутри функций, вроде того, что [ES6 `let` ](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let) . (но не точно, открыть и прочитать ссылки ресурсов)

**********
[javascript](/tags/javascript.md)
