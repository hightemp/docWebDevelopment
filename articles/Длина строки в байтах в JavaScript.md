# Длина строки в байтах в JavaScript

For historical reference or where TextEncoder APIs are [still unavailable](https://caniuse.com/#feat=textencoder) .

If you know the character encoding, you can calculate it yourself though.

 `encodeURIComponent` assumes UTF-8 as the character encoding, so if you need that encoding, you can do,

```
function lengthInUtf8Bytes(str) {
  // Matches only the 10.. bytes that are non-initial characters in a multi-byte sequence.
  var m = encodeURIComponent(str).match(/%[89ABab]/g);
  return str.length + (m ? m.length : 0);
}

```

This should work because of the way UTF-8 encodes multi-byte sequences. The first encoded byte always starts with either a high bit of zero for a single byte sequence, or a byte whose first hex digit is C, D, E, or F. The second and subsequent bytes are the ones whose first two bits are 10. Those are the extra bytes you want to count in UTF-8.

The table in [wikipedia](http://en.wikipedia.org/wiki/UTF-8) makes it clearer

```
Bits        Last code point Byte 1          Byte 2          Byte 3
  7         U+007F          0xxxxxxx
 11         U+07FF          110xxxxx        10xxxxxx
 16         U+FFFF          1110xxxx        10xxxxxx        10xxxxxx
...

```

If instead you need to understand the page encoding, you can use this trick:

```
function lengthInPageEncoding(s) {
  var a = document.createElement('A');
  a.href = '#' + s;
  var sEncoded = a.href;
  sEncoded = sEncoded.substring(sEncoded.indexOf('#') + 1);
  var m = sEncoded.match(/%[0-9a-f]{2}/g);
  return sEncoded.length - (m ? m.length * 2 : 0);
}
```