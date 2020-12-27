---
title: "You don't need xss.rocks/xss.js"
date: 2020-12-27T21:09:00+05:30
draft: false
---

Many newcomers as well as leets focusing on XSS seems to miss out one simple yet powerful thing: data URLs. While finding an XSS, hackers test the vulnerability with some hosted solution like xss.rocks or host their own files. But most of the time, you don't need a hosted javascript file. You can simply use data URLs.

## What is data URL
Data URLs are a special kind of URL defined with a `data` scheme. They work similar to normal URLs and allow you to embed the code in the URL itself. They can be declared with any content type and also can be encoded in base64! Data URL have the following structure:

```http
data:[declarations],[data]
```

You can have multiple declarations delimited by semicolon.

```http
data:text/html;charset=utf-8;base64,PGgxPkhlbGxvLCBXb3JsZCE8L2gxPg==
```

Declarations can be juggled except the content type declaration. The following will fallback to the default content type `text/plain`:

```http
data:charset=utf-8;text/html;base64,PGgxPkhlbGxvLCBXb3JsZCE8L2gxPg==
```

But this is fine:

```http
data:text/html;base64;charset=utf-8,PGgxPkhlbGxvLCBXb3JsZCE8L2gxPg==
```

All fields are optional except for the scheme that is, data. The following will give a blank page:

```http
data:;,
```

## Usefulness

There are times when Content Security Policy is tight enough to block any external javascript but allows data URLs. Note that data URLs are treated as an external resource by browser and will be blocked if Content Security Policy doesn't allows it. So if you can't get XSS via an external hosted javascript file due to CSP, you might be able to get an XSS via data URLs. But it is not always the case. 

As data URLs are more flexible way than the hosted files ie. writing code, saving them and then hosting them is way too work for me atleast. Also, you get base64 encoding on the go. So why not use data URLs where you can. 

### Text in data URL
Most basic format you can get is just text: 

```http
data:;,Hello, World!
```

### HTML in data URL
Writing HTML is just a matter of declaring the content type `text/html`:

```http
data:text/html;,<h1>Hello, World!</h1>
```

But HTML in the URL is ugly. So let's make it base64.

```http
data:text/html;base64,PGgxPkhlbGxvLCBXb3JsZCE8L2gxPg==
```

There is no need of decoding. It is done by browser automagically.

### JavaScript in data URL
As a xss.rocks (or any other hosted javascript files) alternative, you can simply use:

```http
data:text/javascript;,alert('xss')
```

Obviously this might trigger WAF. So simply base64 it.

### Misc

Data URLs are awesome as you can type in your test files on the go without taking efforts to host one. Also, it is not limited to what browser renders. You can have literally any content type and make files on the go!

```http
data:hackers/realm;,{"mongodb_succs":"' || 1==1 %2500"}
```

And yes. Any charset as well:

```http
data:;charset=euc-jp,もし もし
```

If the browser can render, it will:

```http
data:image/svg+xml;,<svg onload="alert('g33z')"></svg>
```

You can use them in script tag:
```html
<script src="data:text/javascript;,alert('lets share your password with the world')"></script>
```

But, links dont work. You might wanna use a hosted file here.
```html
<a href="data:text/html;base64,PGgxPkRvbnQgY2xpY2sgaXQgYnJ1aCEgSnVzdCBraWRkaW5nLi4uPC9oMT4=">Data URLs in a link don't work.</a>
```
