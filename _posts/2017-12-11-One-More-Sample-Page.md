---
title: SQLMAP proxy to shell tools or non-HTTP services.
published: false
---

## The problem:

While solving one of HTB boxes I was introduced to an interesting problem. How to automate the SQL injection exploitation of non-HTTP service? Of course, I did it manually and then after the rush for the first blood, there was time to write a proxy script, which I want to describe below. Because of that this box is not retired yet I can't do it very detailed but you should be able to take advantage of idea itself to create your own custom proxies. Let's start from diving into the information how sqlmap actually works.

## SQLMAP:

You should already know that it can automate the SQL injection attack using requests to HTTP/S sites and some magic :) While passing an interesting HTTP request to it, it can test all parameters, no matter if it is GET or POST or even HTTP header based injection point. But what to do when you know that your non-HTTP service is prone to SQL injection? You can exploit it manually which is recommended for learning sake or use a tool which is dedicated to that service being able to communicate with it and script around your way. 

## Toolbelt:

- sqlmap 1.3.9#stable
- python 3.7

## What I found:

* [Create a simple HTTP server withn Python 3](https://daanlenaerts.com/blog/2015/06/03/create-a-simple-http-server-with-python-3/) -  this post gave me the backbone of my proxy, defin
* [Creating profile with Firefox and Selenium](http://witkowskibartosz.com/blog/selenium-firefox-profile-for-automation.html) - describes how to set up a profile with Firefox needed when installing an add-on



## 

```js
// Javascript code with syntax highlighting.
var fun = function lang(l) {
  dateformat.i18n = require('./lang/' + l)
  return true;
}
```

```ruby
# Ruby code with syntax highlighting
GitHubPages::Dependencies.gems.each do |gem, version|
  s.add_dependency(gem, "= #{version}")
end
```

#### [](#header-4)Header 4

*   This is an unordered list following a header.
*   This is an unordered list following a header.
*   This is an unordered list following a header.

##### [](#header-5)Header 5

1.  This is an ordered list following a header.
2.  This is an ordered list following a header.
3.  This is an ordered list following a header.

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### There's a horizontal rule below this.

* * *

### Here is an unordered list:

*   Item foo
*   Item bar
*   Item baz
*   Item zip

### And an ordered list:

1.  Item one
1.  Item two
1.  Item three
1.  Item four

### And a nested list:

- level 1 item
  - level 2 item
  - level 2 item
    - level 3 item
    - level 3 item
- level 1 item
  - level 2 item
  - level 2 item
  - level 2 item
- level 1 item
  - level 2 item
  - level 2 item
- level 1 item

### Small image

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)


### Definition lists can be used with HTML syntax.

<dl>
<dt>Name</dt>
<dd>Godzilla</dd>
<dt>Born</dt>
<dd>1952</dd>
<dt>Birthplace</dt>
<dd>Japan</dd>
<dt>Color</dt>
<dd>Green</dd>
</dl>

```
Long, single-line code blocks should not wrap. They should horizontally scroll if they are too long. This line should be long enough to demonstrate this.
```

```
The final element.
```
