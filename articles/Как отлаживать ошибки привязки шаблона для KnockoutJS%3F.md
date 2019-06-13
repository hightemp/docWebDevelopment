# Как отлаживать ошибки привязки шаблона для KnockoutJS?

Одна вещь, которую я делаю довольно часто, когда возникает проблема с тем, какие данные доступны в определенной области, - это заменить шаблон / раздел чем-то вроде:

```
<div data-bind="text: ko.toJSON($data)"></div>
```

Если вы используете Chrome для разработки, есть действительно отличное расширение (с которым я не связан), называемое [Knockoutjs context debugger](https://chrome.google.com/webstore/detail/knockoutjs-context-debugg/ oddcpmchholgcjgjdnfjmildmlielhof), который показывает контекст привязки непосредственно на панели «Инструменты разработчика».

**********
[javascript](/tags/javascript.md)
