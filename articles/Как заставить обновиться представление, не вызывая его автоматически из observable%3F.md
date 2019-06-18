# Как заставить обновиться представление, не вызывая его автоматически из observable?

В некоторых случаях может быть полезно просто удалить привязки, а затем повторно применить:

```javascript
ko.cleanNode(document.getElementById(element_id))
ko.applyBindings(viewModel, document.getElementById(element_id))
```