# API CLIPBOARD УСКОРЕННЫЙ КУРС

### Горячая клавиша, все знают

Не каждый предпочитает заказывать компьютер с помощью горячих клавиш. Существует большая группа пользователей и программистов, которые полагаются на графические меню и кнопки. Однако, если есть одна комбинация горячих клавиш, которую все знают, это Cmd + C, Cmd + V. И да, конечно Ctrl + C, Ctrl + V в Windows. ** Но что на самом деле происходит, когда вы копируете? **

### Буфер обмена - это буфер данных

Каждая основная операционная система поставляется с «буфером обмена». Это буфер данных для кратковременного хранения, иногда называемый «вставкой-буфером». Новая команда копирования или вырезания заменяет предыдущее значение в буфере. Почти все программы имеют доступ к этому буферу. В противном случае мы были бы раздражены, когда ничего не происходит, когда мы нажимаем Cmd + V.

### Автоматическое копирование сложного текста может быть хорошим UX

Часто на сайтах нам нужно подарить пользователю специальный код или текст. Этот текст обычно длинный и прямо-таки выглядит как бред. Пользователь, скорее всего, скопирует его сам. Однако, что если они пропустят характер в начале или в конце? Что если они захватят слишком много пробелов, а вы не обрезаете? Что если они просто не понимают, что копировать? Там слишком много места для ошибок. Хорошим решением является автоматическое копирование этого текста в буфер обмена. ** Но там, где есть удобство, есть проблемы. **

### Буфер обмена имеет проблемы с безопасностью

Нет никакого беспокойства, когда пользователь полностью контролирует буфер обмена. Однако что происходит, когда приложение или программа имеет программный доступ к буферу обмена? Могу поспорить, вы можете начать представлять некоторые плохие сценарии. Что если приложение тайно опрашивает ваш буфер обмена и отправляет результаты на сервер? Это потенциально конфиденциальная информация, находящаяся под угрозой, такая как пароли, банковские счета или любой длинный токен безопасности, который вы никогда не хотели вводить самостоятельно. ** Как с этим бороться? **

### Безопасность делает копирование и вставку странным в Интернете

Копирование и вставка имеет странную историю в Интернете. Там нет реального стандарта. Большинство браузеров остановились на `document.execCommand ('copy')` и `document.execCommand ('paste')`. Хотя это кажется достаточно простым, есть некоторая (оправданная) странность.

```js
const result = document.executeCommand('copy');
```

Если это ваша первая кисть с `document.execCommand ('copy')`, вам может быть интересно _"Где текстовый параметр?"_. Возможно, вы все взволнованы, попробовали запустить `document.execCommand('paste')` и увидели, что он абсолютно ничего не делает.

Без некоторых документов вы не можете просто указать, какой текст **вы** хотите добавить в буфер обмена. И вы, конечно, не можете просто получить доступ к тому, что когда-либо было в буфере обмена в то время. Есть некоторые правила, которым вы должны следовать.

### Правила доступа к буферу обмена с executeCommand('copy')

Если вы хотите скопировать текст withdocument.execCommand ('copy'):

1. Контент должен жить в DOM.
2. Пользователь должен вызвать событие (Firefox)
3. Скопированный текст является выбранным пользователем текстом

Chrome не требует запускаемого пользователем события, но требуется в Firefox.

index.js

```js
const button = document.querySelector('#btnCopy');
button.addEventListener('click', event => {
  console.log(document.execCommand('copy'));
});
```

Что если вы хотите скопировать текст, которого нет в DOM? **Вы можете ввести его.**

index.js

```js
const button = document.querySelector('#btnCopy');
button.addEventListener('click', event => {
  const text = 'Hey look, this text is now in the DOM!';
  const input = document.createElement('input');
  document.body.appendChild(input);
  input.value = text;
  input.focus();
  input.select();
  console.log(document.execCommand('copy'));
});
```

Если ваш текст не в DOM, то сделайте это в DOM. Приведенный выше фрагмент создает элемент ввода, добавляет его в DOM, вставляет текст в элемент, фокусирует его, выделяет и, наконец, копирует.

**Что произойдет, если вы не добавите дочерний элемент в DOM, как в** document.body.appendChild (input)? Текст не скопирован. Это не может быть просто отдельный узел DOM, он должен быть в DOM.

Если вы хотите читать напрямую из буфера обмена с помощью document.execCommand('paste'), вы не можете. Это не работает на Chrome, Firefox, Edge или IE. Чтобы получить данные из буфера обмена, вы должны прослушать событие вставки. (Что не поддерживается в IE. История с буфером обмена IE также странная. Это свойство окна и требует разрешения, но давайте не будем вдаваться в подробности).

index.js

```js
document.addEventListener('paste', event => {
  const text = event.clipboardData.getData('text/plain');
  console.log(text);
});
```

Действие вставки пользователя вызывает [ClipboardEvent](https://www.w3.org/TR/clipboard-apis/#clipboardeventinit-clipboarddata). В случае события у вас есть доступ к объекту clipboardDataproperty, который является объектом [DataTransfer](https://html.spec.whatwg.org/multipage/dnd.html#datatransfer). CallinggetData() с правильным форматом возвращает скопированный текст. Хорошо, извини. Это было много ссылок на спецификации.

Это здорово, потому что у вас нет незарегистрированного доступа к буферу обмена пользователя. Пользователь должен сделать вставку. Тем не менее, есть потенциальная проблема. Вызов getData() является синхронным.

Это создает проблему для вас, разработчика. Что если они вставят массивное изображение в кодировке base64? Или какой-то другой безумно большой объем данных? Что делать, если вы интенсивно обрабатываете эти вставленные данные? Это может заблокировать основную ветку страницы, фактически заморозив страницу для вашего пользователя.

К счастью, есть перспективное решение. Это совершенно новый способ чтения и записи данных из буфера обмена.

### API-интерфейс буфера обмена Async(The Async Clipboard API)

Этот новый API имеет несколько улучшений по сравнению с нашим старым другом document.execCommand('copy').

* Отдельные команды для чтения и записи из буфера обмена.
* Асинхронный доступ на основе обещаний к буферу обмена.
* Разрешение на основе. Пользователь должен предоставить разрешение.
* Не требует пользовательского события для запуска.

**Примечание:** Пока это только Chrome. Этот API может быть изменен.

Этот новый API доступен в объекте Navigator: navigator.clipboard

index.js

```js
navigator.clipboard.writeText('whatever you need to copy').then(() => {
  console.log('Text copied')!            
});
```

Пользователю предлагается предоставить разрешение при первом запуске этой команды. Поскольку этот API асинхронный, основной поток понятен, пока мы ждем разрешения пользователя. Если бы API был синхронным, мы бы облажались.

Для чтения из буфера обмена используйте readText():

index.js

```js
navigator.clipboard.readText().then(text => {
  console.log(text);     
});
```

Как и раньше, если это первый раз, пользователь должен предоставить разрешение, прежде чем вы сможете получить доступ к буферу обмена. Помните, что значению буфера обмена трудно доверять, и данные могут быть конфиденциальными. Это все еще хорошая идея, чтобы попросить пользователя вставить значение в случае необходимости.

index.js

```js
document.addEventListener('paste', async event => {
  const text = await navigator.clipboard.readText();
  console.log(text);
});
```

Этот пример является большим улучшением по сравнению с синхронным методом передачи getData(). Пользователь должен предоставить разрешение, и у вас есть асинхронный доступ. Вы лучше защищены от большого количества текста, и легче выполнять более интенсивную обработку вставленного текста.

### Only the active tab has access to the clipboard

Users keep tabs open forever. Having unfettered access to the keyboard is still a bad idea. You could openexample.comin a tab and leave it there for a week._Don't act like you haven't had a tab open for a week._During that time you'll have all sorts of data go in and out of your Clipboard. What ifexample.compolled thenavigator.clipboard.readText()method? That would be mighty dangerous, but there's a safeguard.

Your site no longer has access to the Clipboard when the user switches to another tab. This is a great precaution, but it comes with an annoying debugging problem. When you open up the DevTools in Chrome, the DevTools itself becomes the active tab. ThereadText()orwriteText()promise will reject and you'll be annoyed and be all like_"This is why I don't use brand-new APIs"_. The trick is to defer the call until you can click back into the tab.

index.js

```js
setTimeout(async () => {
  const text = await navigator.clipboard.readText();
  console.log(text);
}, 4000);
```

This isn't much better, but it'll work. This quirk isn't the only drawback of the Async Clipboard API.

### The Async Clipboard API doesn't provide the user-selected text

Usingdocument.execCommand('copy')will automatically copy the user's selected text. If you need that functionality, you'll have to get the selected text yourself and pass it in towriteText(). That will require a combination of the[Selection API](https://developer.mozilla.org/en-US/docs/Web/API/Selection), the[Range API](https://developer.mozilla.org/en-US/docs/Web/API/Range), and some DOM traversal. You'll have to deal with combining text nodes across a range of DOM elements, and that's not fun.

### Learn more about the Async Clipboard API

Jason Miller, who is a legend,[wrote an amazing article on the Async Clipboard API](https://developers.google.com/web/updates/2018/03/clipboardapi). It's basically the standard resource at the moment.

### Copy and paste responsibly

Clipboard access is a great tool for user experience, but it has its thorns. Some users carry sensitive data and some users bring malicious data. Make sure you handle user's data responsibly and prepare yourself for those nasty paste events.

