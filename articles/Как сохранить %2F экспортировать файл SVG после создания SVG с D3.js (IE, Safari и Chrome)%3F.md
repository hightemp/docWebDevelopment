# Как сохранить / экспортировать файл SVG после создания SVG с D3.js (IE, Safari и Chrome)?

Есть 5 шагов. Я часто использую этот метод для вывода встроенного SVG.

1. получить встроенный элемент SVG для вывода.
2. получить исходный код SVG с помощью XMLSerializer.
3. добавить пространства имен svg и xlink.
4. построить схему данных URL svg методом encodeURIComponent.
5. установите для этого URL-адреса атрибут href некоторого элемента «a» и щелкните правой кнопкой мыши по этой ссылке, чтобы загрузить файл SVG.

* * *

```javascript
//get svg element.
var svg = document.getElementById("svg");

//get svg source.
var serializer = new XMLSerializer();
var source = serializer.serializeToString(svg);

//add name spaces.
if(!source.match(/^<svg[^>]+xmlns="http\:\/\/www\.w3\.org\/2000\/svg"/)){
    source = source.replace(/^<svg/, '<svg xmlns="http://www.w3.org/2000/svg"');
}
if(!source.match(/^<svg[^>]+"http\:\/\/www\.w3\.org\/1999\/xlink"/)){
    source = source.replace(/^<svg/, '<svg xmlns:xlink="http://www.w3.org/1999/xlink"');
}

//add xml declaration
source = '<?xml version="1.0" standalone="no"?>\r\n' + source;

//convert svg source to URI data scheme.
var url = "data:image/svg+xml;charset=utf-8,"+encodeURIComponent(source);

//set url value to a element's href attribute.
document.getElementById("link").href = url;
//you can download svg file by right click menu.
```

**********
[SVG](/tags/SVG.md)
