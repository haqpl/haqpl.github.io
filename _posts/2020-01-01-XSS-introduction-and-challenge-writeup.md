---
title: XSS introduction and challenge writeup.
published: false
author: Maciej Piechota
---

A few months ago I took place in the XSS challenge organized by @haxel0rd and later was asked about explaining my solutions. The challenge was divided into levels starting from easy to the hard one, each level was about exploiting different XSS context, which was great in terms of learning sake. I will describe each solution and some theory behind them, so let's dive in.

## The challenge:

https://twitter.com/haxel0rd/status/1116822993085325312

Loging in itself is also a baby SQL injection challenge - `admin' OR 1=1-- t` - will do the job bypassing the authentication.

## Fast XSS methodology:

First we want to find out if and where our input is reflected in the attacked page(this is so called `injection context`), then we have to check what transformations are being made to our payload by the application, giving us the information how CSS/HTML/JavaScript sensitive characters are treated and what possibilities to inject malicious code are left unsecured. To do this in one request let's use the XSS probe:
`aaaaaa'">xsshere`
I type this to the interesting input field, submit, and then check the response for `xsshere` string as shown in Lvl01 below. 
Probe is sent via HTTP/S proxy like Burp and with opened Developer Console in browser to observe JavaScript errors.

## Lvl01:

{% include figure.html file="/assets/lvl01.png" alt="/assets/lvl01.png" max-width="500px" number="1" caption="Localizing the inection contenxt." %}
                                                                                                                                  Ok, we see here that our probe broke the rendering of the HTML, this is always a good sign for pentester and worse for a developer :) You can read it as: some of characters from the probe are not encoded properly before returning them to the client and interpreted by a browser as a legit code, having the influence on the final look/JavaScript workflow of the page.

```html
<input type="text" class="form-control input-lg" id="search-church" id="xss" value='aaaaaa'">xsshere' name="xss" placeholder="xss">                                                                                                                               
```                                                                                                                                  
                                                                                                                                  this is called `HTML attribute injection context`. The HTML attributes are closed by characters: `'` or `"` so injecting one as a payload will close the attribute and allow us to add new attribute or just close HTML tag with `>` character. This happened here, `input` tag was closed, so we can inject a new tag. To achive our goal - execute `alert(1)` we have to inject `<script>alert(1)</script>`.

Final payload:

Add dummy attribute value, close attribute, close input tag, add script tag with JavaScript code.

`x'><script>alert(1)</script>` which translates to:

```html
<input type="text" class="form-control input-lg" id="search-church" id="xss" value='x'><script>alert(1)</script>' name="xss" placeholder="xss">
```

{% include figure.html file="/assets/alert.png" alt="/assets/alert.png" max-width="500px" number="1" caption="Profit." %}

## Lvl02:

{% include figure.html file="/assets/lvl2.png" alt="/assets/lvl2.png" max-width="500px" number="1" caption="Level 02." %}

```html
<div class='aaaaaa'&quot;&gt;xsshere'>like Aldus PageMaker including versions of Lorem Ipsum.</div>
```

Our probe ended up also in the HTML attribute context but this time the `>` character is properly encoded to its HTML equivalent: `&gt;` which makes it impossible to close input tag but it doesn't mean we can't inject a new attributes. To achieve our goal we have to use HTML events which as value takes JavaScript.

Solution:

`x' onmouseover=alert(1) x`

## Lvl03:

https://hackme.obscurity.app/dashboard/?page=xss&level=3

Sometimes the input to the application is passed via URL, we have to identify the parameters which are used in the JavaScript source code (static analysis).

{% include figure.html file="/assets/lvl3.png" alt="/assets/lvl3.png" max-width="500px" number="1" caption="Level 03." %}

`$('#hmmm').append("<li ${_text}>Hello world!</li>");`

this line is vulnerable, again HTML attribute context but we can't use any of interesting characters because of filtering:

```js
function escapeOutput(toOutput){
    return toOutput.replace(/\&/g, '&amp;')
        .replace(/\</g, '&lt;')
        .replace(/\>/g, '&gt;')
        .replace(/\"/g, '&quot;')
        .replace(/\'/g, '&#x27')
        .replace(/\//g, '&#x2F');
}
```

luckily we do not have to use them. Browsers are so kind that they try to fix "mistakes" for us:

Payload:
`xss=onmouseover=alert(1)`

the value `alert(1)` of event is automatically enclosed with `"` character.

{% include figure.html file="/assets/lvl3_1.png" alt="/assets/lvl3_1.png" max-width="500px" number="1" caption="Level 03 - Payload." %}

## Lvl04:

This level was solved with unintended solution and introduces to us the next injection context - JavaScript context.

{% include figure.html file="/assets/lvl4.png" alt="/assets/lvl4.png" max-width="500px" number="1" caption="Level 04 - JavaScript context." %}



## Credits:

@haxel0rd
