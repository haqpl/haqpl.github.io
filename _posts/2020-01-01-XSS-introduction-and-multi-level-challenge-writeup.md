---
title: XSS introduction and multi-level challenge writeup.
published: true
author: Maciej Piechota
---

A few months ago I took place in multi-level XSS challenge organized by @haxel0rd and later was asked about explaining my solution. The challenge was divided into 10 levels with increasing difficulty. Almost each level was about exploiting different XSS context, which was great in terms of learning sake. I will describe each solution and some schematics behind them, so let's dive in.

## The challenge:

<center><twitter-widget class="twitter-tweet twitter-tweet-rendered" id="twitter-widget-0" style="position: static; visibility: visible; display: block; transform: rotate(0deg); max-width: 100%; width: 500px; min-width: 220px; margin-top: 10px; margin-bottom: 10px;" data-tweet-id="1116822993085325312"></twitter-widget>
  <blockquote class="twitter-tweet" data-link-color="#E95F28">
    <p lang="en" dir="ltr">Can you beat all 10 levels..?
#XSS #WAF #FilterEvasion #Challenge, from Beginner => Hard, pure XSS, no fantasy vectors. Will post first 7 solvers on my twitter. The challenge starts at login (no-xss), good luck. #Hacking #HackIt #CTF https://u.nu/hackit CodedBy: @ObscurityApp
        💪 <a href="https://t.co/VLSJ4mPZqy">pic.twitter.com/VLSJ4mPZqy</a></p>&mdash; haxel0rd (@haxel0rd) <a href="https://twitter.com/haxel0rd/status/1116822993085325312">April 12, 2019</a></blockquote>
</center>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Logging into the challenge was an entry obstacle itself. It was a simple SQL Injection - `admin' OR 1=1-- t` - did the job, bypassing the authentication.

## Brief XSS methodology:

To start exploiting XSS you first have to find out if and where our input is reflected on the attacked page(this is so-called `injection context`). Then, when we found that, we have to check what transformations are being made to our payload by the application, giving us the information how CSS/HTML/JavaScript sensitive characters are treated and what possibilities to inject malicious code are left unsecured. To do this in one request let's use an XSS probe:
`aaaaaa'">xsshere`
I type this to the interesting input field, submit, and check the response for `xsshere` string as shown in Lvl01 below.
The probe is sent via HTTP/S proxy like Burp and with opened Developer Console in browser to observe JavaScript errors.

## Lvl01:

By following the steps from “Brief XSS methology” we can see here that our probe broke the rendering of the HTML - this is usually a good sign for pentester and not that good for a developer :) You can read it as: some of the probe’s characters  are not properly encoded/escaped  before returning them to the client and therefore interpreted by a browser as a legit code, having an influence on the final look/JavaScript workflow of the page.

{% include figure.html file="/assets/lvl01.png" alt="/assets/lvl01.png" max-width="500px" number="1" caption="Localizing the injection context." %}
                                                                                                                                  
```html
<input type="text" class="form-control input-lg" id="search-church" id="xss" value='aaaaaa'">xsshere' name="xss" placeholder="xss">
```

`’` character closed an attribute, `>` closed the tag and the rest was rendered as text.

We have here `HTML attribute injection context`. The HTML attributes can be enclosed by characters: `'`, `"` or they can appear without anything - up to the first white character so injecting `'` will close the attribute and allow us to add a new one or just close HTML tag with `>` character. This happened in the example above - `input` tag was closed, so we can inject a new tag. To achieve our goal - execute `alert(1)` we have to inject `<script>alert(1)</script>`.

Final payload:

Add a dummy attribute value inside an input, close the attribute, close the input tag, add a script tag with JavaScript code.

`x'><script>alert(1)</script>` which translates to:

```html
<input type="text" class="form-control input-lg" id="search-church" id="xss" value='x'><script>alert(1)</script>' name="xss" placeholder="xss">
```

{% include figure.html file="/assets/alert.png" alt="/assets/alert.png" max-width="500px" number="2" caption="Profit." %}

## Lvl02:

In level 2, the probe from Lvl01 also ended up in the `HTML attribute context` but this time the `>` character is properly escaped to its HTML equivalent: `&gt;` which makes it hard to escape from the context but it doesn't mean we can't inject new attributes. To achieve our goal we have to use HTML events which as value takes JavaScript.

{% include figure.html file="/assets/lvl2.png" alt="/assets/lvl2.png" max-width="500px" number="3" caption="Level 02." %}

```html
<div class='aaaaaa'&quot;&gt;xsshere'>like Aldus PageMaker including versions of Lorem Ipsum.</div>
```
Solution:

`x' onmouseover=alert(1) x`

and this is how our payload looks in HTML:

```html
<div class='x' onmouseover=alert(1) x''>like Aldus PageMaker including versions of Lorem Ipsum.</div>
```
`alert(1)` will be executed after triggering the `onmouseover` event. It is important to take care of the last `'` character simply adding a dummy `x` attribute.

## Lvl03:

Sometimes the input to the application is passed via URL, we have to identify the parameters which are used in the JavaScript source code(static analysis).

{% include figure.html file="/assets/lvl3.png" alt="/assets/lvl3.png" max-width="500px" number="4" caption="Level 03." %}

`$('#hmmm').append("<li ${_text}>Hello world!</li>");`

the above line is vulnerable to XSS, again the `HTML attribute context` but we can't use any of interesting characters because of filtering:

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

Luckily we do not have to use them. Browsers are so kind that they try to fix "mistakes" for us:

Payload:
`xss=onmouseover=alert(1)`

the value `alert(1)` of the event is automatically enclosed with `"` character.

{% include figure.html file="/assets/lvl3_1.png" alt="/assets/lvl3_1.png" max-width="500px" number="5" caption="Level 03 - Payload." %}

## Lvl04:

This level was solved with an unintended solution and introduces the next injection context - `JavaScript context`.

{% include figure.html file="/assets/lvl4.png" alt="/assets/lvl4.png" max-width="500px" number="6" caption="Level 04 - JavaScript context." %}

Developer console shows the following JavaScript error:

{% include figure.html file="/assets/lvl4_1.png" alt="/assets/lvl4_1.png" max-width="500px" number="7" caption="Level 04 - JavaScript error." %}

This tells us that the execution of the JavaScript was interrupted because of the injected payload. In a scenario like this we usually want to:

- Close all opened strings, parentheses, remember that the JavaScript before injection must be "satisfied".
- Add `;` as we want to start new instruction.
- alert(1)
- Deal with code that was there before injection making it as a comment with `//`.

Solution:
`xsshere'); alert(1)//`

Intended solution:

{% include figure.html file="/assets/lvl4_2.png" alt="/assets/lvl4_2.png" max-width="500px" number="8" caption="Level 04 - Intended solution." %}

## Lvl05:

This level is very similar to the previous one - we are once again in the `JavaScript context`.

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

But there is a filter - we can't start a new `script` tag. There are three strings literals in JavaScript: `'`, `"` and `` ` ``. So following schematics from the prevoius level, we craft a payload like: ``xsshere ` ; alert(1)//`` and this is the solution for this level.




## TBC

The rest of 10 levels to be continued.

## Credits:

@haxel0rd


