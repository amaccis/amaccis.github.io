---
title: "Size Matters"
date: 2023-04-25
draft: false
tags: ["English", "PHP", "Node.js"]
---

![tape measure](/images/posts/tape_measure.jpg)

Since I was curious about **PHP** [Foreign Function Interface](https://www.php.net/manual/en/book.ffi.php), after the release of PHP 7.4.0 I decided to write the library 
[PHP Stemmer](https://github.com/amaccis/php-stemmer), a simple PHP interface to the [Snowball stemming algorithms](https://snowballstem.org/).

After some time I thought I could take advantage of that experience for having some more fun, so I thought that writing another interface to the Snowball stemming algorithms
(this time with another technology, **Node.js**) could be the right choice and I wrote [Node Stemmer](https://github.com/amaccis/node-stemmer).

With Node.js in order to have something like PHP FFI, you can choose between a few possibilities and I chose [node-ffi-napi](https://github.com/node-ffi-napi/node-ffi-napi).

During the process I realized that Node.js FFI was more difficult to handle for me than PHP FFI, perhaps because using [ref](https://github.com/TooTallNate/ref) to turn buffer
instances into C pointers and vice-versa assumes you are pretty much aware of what you are doing and, at the beginning, I certainly was not.

Anyway the interesting thing I want to stress in this post is how the string size matters when you are handling **C** pointers to strings.

Let's say you have to deal with a header like the one below.
```c
struct sb_stemmer;

typedef unsigned char sb_symbol;

const sb_symbol * sb_stemmer_stem(struct sb_stemmer * stemmer, const sb_symbol * word, int size);
```
Putting aside all the other things (that I quoted only for giving context) let's focus on the last argument of `sb_stemmer_stem()`, that is `size`, an `int` that in this case 
represents the size in bytes of the word you want to get the stem back.

The first 128 Unicode code points are encoded as 1 byte in [**UTF-8**](https://en.wikipedia.org/wiki/UTF-8) and this may be confusing, programming in PHP, especially if you are 
used to handle only italian words, because using this language, for a string the length in characters is often the same as its length in bytes.

So, let's use the portuguese!

```php
<?php 

$string = "atribuição";

echo strlen($string); // 12

echo iconv_strlen($string); // 10
```

As you can see by reading the [documentation](https://www.php.net/manual/en/function.strlen.php) `strlen()` returns the length in bytes of a string. So if you need the length
in characters, you need to use [`iconv_strlen()`](https://www.php.net/manual/en/function.iconv-strlen.php).

But what about **JavaScript**?

```typescript
const string = "atribuição";

console.log(string.length); // 10

console.log(new Blob([string]).size); // 12
```

It sounds a little counterintuitive compared to PHP (even if probably it is more intuitive in general terms...) but if you try to get the 
[`length` of a JavaScript string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/length), you'll get its length in characters.
If you need its length in bytes, you need to rely on [`Blob()`](https://developer.mozilla.org/en-US/docs/Web/API/Blob).

