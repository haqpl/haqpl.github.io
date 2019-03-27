---
title: Automation of KNOXSS extension using Selenium and Python
published: true
author: Maciej Piechota
---

I'm not a programmer but I like to automate some boring stuff I do every day using Python. That's why I decided to write a few lines of code and give life to semi-automatic XSS scanner of my choice which is Knoxss.
Knoxss comes to you as an extension. The author recommends using Gecko based browsers so I chose Firefox in developer edition because it allows you to load extensions using Web driver - Selenium.

### The problem:

Knoxss is a semi-automatic tool which means that you have to manually browse a site you want to scan it for XSS vulnerabilities. The add-on deserves automation on its own.

### Toolbelt:

- geckodriver 0.23.0 ( 2018-10-04)
- Mozilla Firefox 67.0b4 Developer Edition
- Selenium 3.141.0
- KNOXSS Add-on 1.2.0
- Python 3.6.5
- Visual Studio Code :love:

### KNOXSS specification:

Author of this extensions recommends using Gecko based browser so obvious choice would be Firefox.


### Credits:

@rodoassis
@razorjack

