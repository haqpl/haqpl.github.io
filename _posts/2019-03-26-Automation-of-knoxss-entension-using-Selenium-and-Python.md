---
title: Automation of KNOXSS extension using Selenium and Python
published: true
author: Maciej Piechota
---

I'm not a programmer but I like to automate some boring stuff I do every day using Python. That's why I decided to write a few lines of code and give life to semi-automatic XSS scanner of my choice which is KNOXSS.
KNOXSS comes to you as an extension. The author recommends using Gecko based browsers so I chose Firefox in developer edition because it allows you to load extensions using Web driver - Selenium.

## The problem:

Knoxss is a semi-automatic tool which means that you have to manually browse a site you want to scan it for XSS vulnerabilities. The add-on deserves automation on its own.

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

## What I found:

* [Firefox extensions with selenium](https://intoli.com/blog/firefox-extensions-with-selenium/) - I started with this blog post, which gave me an idea how to load an Firerfox extension using Selenium
* [Creating profile with Firefox and Selenium](http://witkowskibartosz.com/blog/selenium-firefox-profile-for-automation.html) - describes how to setup a profile with Firefox needed when installing an add-on

## Preparing:

1. Download geckodriver - [Geckodriver release](https://github.com/mozilla/geckodriver/releases)
2. Download Firefox Developer Edition - [Firefox Developer Edition](https://www.mozilla.org/pl/firefox/developer/)
3. Install latest Selenium in your Python environment - `pip install selenium --user`
4. Download KNOXSS Pro Add-on - [KNOXSS](https://knoxss.me/)
5. Run `python knoxss_automation.py -u URL -c COOKIES`


## Credits:

[@rodoassis](https://twitter.com/rodoassis)
[@razorjack](https://razorjack.net)

