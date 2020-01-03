---
title: XSS introduction and challenge writeup.
published: true
author: Maciej Piechota
---

A few months ago I took place in the XSS challenge organized by @haxel0rd and later was asked about explaining my solutions. The challenge was divided into levels starting from easy to the hard one, each level was about exploiting different XSS context, which was great in terms of learning sake. I will describe each solution and some theory behind them, so let's dive in.

## The challenge:

<twitter-widget class="twitter-tweet twitter-tweet-rendered" id="twitter-widget-0" style="position: static; visibility: visible; display: block; transform: rotate(0deg); max-width: 100%; width: 500px; min-width: 220px; margin-top: 10px; margin-bottom: 10px;" data-tweet-id="1116822993085325312"></twitter-widget>

Loging in itself is also a baby SQL injection challenge - `admin' OR 1=1-- t` - will do the job bypassing the authentication.

## Fast XSS methodology:

First, we want to find out if and where our input is reflected in the attacked page(this is so-called `injection context`), then we have to check what transformations are being made to our payload by the application, giving us the information how CSS/HTML/JavaScript sensitive characters are treated and what possibilities to inject malicious code are left unsecured. To do this in one request let's use the XSS probe:
`aaaaaa'">xsshere`
I type this to the interesting input field, submit, and then check the response for `xsshere` string as shown in Lvl01 below. 
Probe is sent via HTTP/S proxy like Burp and with opened Developer Console in browser to observe JavaScript errors.

## Lvl01:

Ok, we see here that our probe broke the rendering of the HTML, this is always a good sign for pentester and worse one for a developer :) You can read it as: some of characters from the probe are not encoded properly before returning them to the client and interpreted by a browser as a legit code, having the influence on the final look/JavaScript workflow of the page.

{% include figure.html file="/assets/lvl01.png" alt="/assets/lvl01.png" max-width="500px" number="1" caption="Localizing the inection contenxt." %}
                                                                                                                                  
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

{% include figure.html file="/assets/alert.png" alt="/assets/alert.png" max-width="500px" number="2" caption="Profit." %}

## Lvl02:

Our probe ended up also in the HTML attribute context but this time the `>` character is properly encoded to its HTML equivalent: `&gt;` which makes it impossible to close input tag but it doesn't mean we can't inject new attributes. To achieve our goal we have to use HTML events which as value takes JavaScript.

{% include figure.html file="/assets/lvl2.png" alt="/assets/lvl2.png" max-width="500px" number="3" caption="Level 02." %}

```html
<div class='aaaaaa'&quot;&gt;xsshere'>like Aldus PageMaker including versions of Lorem Ipsum.</div>
```
Solution:

`x' onmouseover=alert(1) x`

## Lvl03:

Sometimes the input to the application is passed via URL, we have to identify the parameters which are used in the JavaScript source code (static analysis).

{% include figure.html file="/assets/lvl3.png" alt="/assets/lvl3.png" max-width="500px" number="4" caption="Level 03." %}

`$('#hmmm').append("<li ${_text}>Hello world!</li>");`

this line is vulnerable, again HTML attribute context but we can't use any of interesting characters because of filtering:

```javascript
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

{% include figure.html file="/assets/lvl3_1.png" alt="/assets/lvl3_1.png" max-width="500px" number="5" caption="Level 03 - Payload." %}

## Lvl04:

This level was solved with an unintended solution and introduces to us the next injection context - JavaScript context.

{% include figure.html file="/assets/lvl4.png" alt="/assets/lvl4.png" max-width="500px" number="6" caption="Level 04 - JavaScript context." %}

Developer console shows the JavaScript error: 

{% include figure.html file="/assets/lvl4_1.png" alt="/assets/lvl4_1.png" max-width="500px" number="7" caption="Level 04 - JavaScript error." %}

which tells us that the execution of the JavaScript was interrupted because of our injected payload. The steps to do in that case are usually the same:

1. Close all opened strings, parentheses, remember that the JavaScript before injection must be "satisfied".
2. Add `;` as we want to start new instruction.
3. alert(1)
4. Deal with code that was there before injection making it as a comment with `//`.

Solution:
`xsshere'); alert(1)//`

Intended solution:

{% include figure.html file="/assets/lvl4_2.png" alt="/assets/lvl4_2.png" max-width="500px" number="8" caption="Level 04 - Intended solution." %}

## Lvl05:

This is very similar to the previous level.

We're in the JavaScript context.

{% include figure.html file="/assets/lvl5.png" alt="/assets/lvl5.png" max-width="500px" number="9" caption="Level 05 - JavaScript variable context." %}


```javascript
function escapeOutput(toOutput){
    return toOutput.replace(/\&/g, '&amp;')
        .replace(/\</g, '&lt;')
        .replace(/\>/g, '&gt;')
        .replace(/\"/g, '&quot;')
        .replace(/\'/g, '&#x27')
        .replace(/\//g, '&#x2F');
}

let input = `aaaaa'">xsshere`;
$("#hello-xss").append(`Nothing intresting found? Input = '${escapeOutput(input)}'`);
```

but there is a filter, we can't start a new `script` tag. JavaScript implements three characters which identify strings: `'`, `"` and `&grave;`. So payload like: `xsshere&grave;; alert(1)//` is solution for this level.




## TBC

## Credits:

@haxel0rd
