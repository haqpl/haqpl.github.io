---
title: Automation of KNOXSS extension using Selenium and Python
published: true
author: Maciej Piechota
---

I like to automate some boring stuff I do every day using Python. That's why I decided to write a few lines of code and give life to a semi-automatic XSS scanner which is KNOXSS.
KNOXSS comes to you as an extension. The author recommends using Gecko based browsers so I chose Firefox in developer edition because it allows you to load extensions using a Web driver  - Selenium. Please notice that I'm not a software architect :) My job was to make it work and solve the problem, if you have any ideas how to improve the code or add any functionalites, please let me know in comments.

## The problem:

KNOXSS is a semi-automatic tool which means that you have to manually browse a site you want to scan it for XSS vulnerabilities. I think the add-on deserves automation on its own.

## Toolbelt:

- geckodriver 0.23.0 ( 2018-10-04)
- Mozilla Firefox 67.0b4 Developer Edition
- Selenium 3.141.0
- KNOXSS Add-on 1.2.0
- Python 3.6.5
- Visual Studio Code :love:

## KNOXSS specification:

As we can read on the [knoxss.me](https://knoxss.me) page:

>it supports source and DOM based reflected XSS, although by chance a stored or a more complex DOM-based case may arise if there's also a reflection in response. Except for demo plan (below), it also drops (in certain cases) a XSS payload designed to send an email report to KNOXSS user with info about the environment where it was triggered (in scenarios where such vulnerability exists) hence also being able to find blind and stored XSS cases in this way.

## What is Selenium:

Selenium automates browsers. It is widely used by automation testers but can be used in security as well. Someone can say: I can do the same with wget/curl. But Selenium can render the whole page including JavaScript and CSS, that is useful when talking about DOM-based XSS'es. Add to it a method for detecting JavaScript alerts, Radamsa for mutation of parameters values, and you have almost a full dynamic scanner, I know that almost makes a difference :)

## What I found:

* [Firefox extensions with selenium](https://intoli.com/blog/firefox-extensions-with-selenium/) -  I started with this blog post, which gave me an idea of how to load a Firefox extension using Selenium
* [Creating profile with Firefox and Selenium](http://witkowskibartosz.com/blog/selenium-firefox-profile-for-automation.html) - describes how to set up a profile with Firefox needed when installing an add-on

## Instalation:

1. Download latest geckodriver - [Geckodriver release](https://github.com/mozilla/geckodriver/releases) - it should be placed in `/usr/bin/` or in $PATH variable.

`wget https://github.com/mozilla/geckodriver/releases/download/v0.24.0/geckodriver-v0.24.0-linux64.tar.gz && tar -zxvf geckodriver-v0.24.0-linux64.tar.gz && cp geckodriver /usr/bin/`

2. Download Firefox Developer Edition - [Firefox Developer Edition](https://www.mozilla.org/pl/firefox/developer/)
3. Install latest Selenium in your Python environment.

`pip install selenium --user`

4. Download KNOXSS Pro add-on - [KNOXSS](https://knoxss.me/) and unzip the XPI file. 

`unzip knoxss_add_on-1.2.0-fx.xpi -d knoxss`

5. Locate the background script `index.js` and edit __msgKnoxss__ function, it should look like that below:

```javascript
function msgKnoxss(text) {
   text = text.replace(/(\r\n|\n|\r)/gm, " ");
   browser.tabs.executeScript(null, { code: "var myEvent = new CustomEvent('knoxss_status',{'detail': '"+text+"'}); document.dispatchEvent(myEvent); myEvent.preventDefault();"});
   
   browser.notifications.create({
     "type": "basic",
     "iconUrl": browser.extension.getURL("icons/k.png"),
     "title": "KNOXSS Msg Service",
     "message": text
   });
}
```
6. Login to the KNOXSS service and get your Cookies, it has to be passed in CLI.
7. Run `python3 automate_knoxss.py -u target_url -c cookies_string -f firefox_binary -a modified_knoxss_dir -t timeout`

## What is going on here:

Like I wrote in my previous post, explaining modifications: [Communication between Selenium, page script and Firefox extension](Communication-between-Selenium-page-script-and-Firefox-extension). 

The tool supports the basic method of navigating and scraping links of each visited web page starting from that passed in the argument. It compares each found link to the visited ones, looking for proper extensions and checks whether the domain is suitable. It communicates with KNOXSS add-on using JavaScript custom events, that method might be useful for automation testers.

## Parameters description:

`python3 automate_knoxss.py -u "https://target" -c "wordpress_logged_in_...=...; wordpress_sec_...=...; sucuri_cloudproxy_uuid_...=...; wordpress_test_cookie=WP+Cookie+check;" -f /home/firefox/firefox -a knoxss -t 90`

1. `-u` or `--url` - defines the target for the scan
2. `-c` or `--cookies` - defines the session Cookies for logged in user to KNOXSS service
3. `-f` or `--firefox` - defines the location of Firefox Developer edition binary
4. `-a` or `--addon` - defines the location of KNOXSS extension directory, unzipped and modified
5. `-t` or `--timeout` - defines the timout for event

## Repository:

[https://github.com/haqpl/automate_knoxss](https://github.com/haqpl/automate_knoxss)

Please be aware when using this tool the author of KNOXSS scanner will **ban** users who will abuse his service!

## TODO:

1. There is a problem with synchronizing visited pages and extension status because of the nature of the Internet but I will solve it in the next releases of my tool. I hope :)
2. Resume functionality - saving the state of a scan to the file and loading it on request.
3. Summary of scanning results. For now, I recommend to use tool with `script` command to save the results or watch the browser from time to time :)
4. Threading for navigating simultaneously in several browsers

## Credits:

@rodoassis
@razorjack
