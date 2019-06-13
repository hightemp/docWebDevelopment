# В "Knockout" сделать неактивным элемент в "select"

Вы можете создать элементы `option`, используя `foreach`, например:

```html
<select data-bind="value: selected, foreach: options">
    <option data-bind="attr: { disabled: !enabled(), value: value }, text: name"></option>
</select>
```

Образец: [http://jsfiddle.net/rniemeyer/4PuxQ/](http://jsfiddle.net/rniemeyer/4PuxQ/)

Иначе, вот пример привязки `optionsBind` от Michael Best, которая позволит вам связать опцию без использования foreach (использует функциональность `optionsAfterRender`):

```html
<select data-bind="value: selected, options: options, optionsText: 'name', optionsValue: 'value', optionsBind: 'attr: { disabled: !enabled() }`"></select>'
```

Образец: [http://jsfiddle.net/rniemeyer/KxY44/](http://jsfiddle.net/rniemeyer/KxY44/)