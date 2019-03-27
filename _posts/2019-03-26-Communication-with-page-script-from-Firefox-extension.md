---
title: Communication between page script and Firefox extension
published: true
---
### [](#header-3)The problem:

Few days ago I wrote a tool which automates Knoxss extension. In the beginning my tool was not able to communicate with extension so I couldn't read its status and so on. The problem was that every solution I found described communication from HTML page to extension but I needed something oposite. Second problem: I'm not a JavaScript programmer :)

Moreover, I was using Selenium and needed information from extension in my Python code.

### What I found:

* [Communication between HTML and your extension](https://developer.mozilla.org/en-US/docs/Archive/Add-ons/Communication_between_HTML_and_your_extension) - described usage of Custom events which gave me an idea of trying it
* [Can my webdriver script catch a event from the webpage](https://stackoverflow.com/questions/35884230/can-my-webdriver-script-catch-a-event-from-the-webpage) - Catching JavaScript events by Selenium Webdriver

### What I tried:

1. Intercepting Console API entries in Selenium - [Support for Selenium’s logging interface](https://github.com/mozilla/geckodriver/issues/284) - didn't work for me
2. Communicating with alert box (editing extension code and add `alert(status)` somewhere in msgKnoxss function - possible but not elegant.

### Some theory about extensions:

There are three different types of JavaScript code when talking about extensions:

| Name  | Properties |
| ------------- | ------------- |
| Page script  | Code running in context of the web page  |
| Content script  | Part of the extension but running in context of the web page, so called proxy between page and background scripts  |
| Background script  | Logic of the extension, **could not** communicate with page script, what is my purpose  |
