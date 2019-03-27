---
title: Communication between page script and Firefox extension
published: true
---
### [](#header-3)The problem:

Few days ago I wrote a tool which automates Knoxss extension. In the beginning my tool was not able to communicate with extension so I couldn't read its status and so on. The problem was that every solution I found described communication from HTML page to extension but I needed something oposite. Second problem: I'm not a JavaScript programmer :)

Moreover, I was using Selenium and needed information from extension in my Python code.

### What I found:

* [Communication between HTML and your extension](https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Communication_between_HTML_and_your_extension) - described usage of Custom events which gave me an idea of trying it
* traaa

### What I tried:

1. Intercepting Console API entries in Selenium - [Support for Selenium’s logging interface](https://github.com/mozilla/geckodriver/issues/284)
2. Communicating with alert box (editing extension code and add ```alert(status)``` - possible but not elegant.
