# Длина строки в байтах в JavaScript

Для исторической справки или где API TextEncoder [все еще недоступны](https://caniuse.com/#feat=textencoder).

Если вы знаете кодировку символов, вы можете вычислить ее самостоятельно.

 `encodeURIComponent` предполагает UTF-8 в качестве кодировки символов, поэтому, если вам нужна эта кодировка, вы можете сделать,

```javascript
function lengthInUtf8Bytes(str) {
  // Matches only the 10.. bytes that are non-initial characters in a multi-byte sequence.
  var m = encodeURIComponent(str).match(/%[89ABab]/g);
  return str.length + (m ? m.length : 0);
}

```

Это должно работать из-за способа, которым UTF-8 кодирует многобайтовые последовательности. Первый кодированный байт всегда начинается либо с старшего бита нуля для одной последовательности байтов, либо с байта, чья первая шестнадцатеричная цифра - C, D, E или F. Второй и последующие байты - это те, чьи первые два бита равны 10 Это те дополнительные байты, которые вы хотите считать в UTF-8.

Таблица в [wikipedia](http://en.wikipedia.org/wiki/UTF-8) проясняет ситуацию

```javascript
Bits        Last code point Byte 1          Byte 2          Byte 3
  7         U+007F          0xxxxxxx
 11         U+07FF          110xxxxx        10xxxxxx
 16         U+FFFF          1110xxxx        10xxxxxx        10xxxxxx
...

```

Если вместо этого вам нужно понять кодировку страницы, вы можете использовать этот трюк:

```javascript
function lengthInPageEncoding(s) {
  var a = document.createElement('A');
  a.href = '#' + s;
  var sEncoded = a.href;
  sEncoded = sEncoded.substring(sEncoded.indexOf('#') + 1);
  var m = sEncoded.match(/%[0-9a-f]{2}/g);
  return sEncoded.length - (m ? m.length * 2 : 0);
}
```

**********
[javascript](/tags/javascript.md)
