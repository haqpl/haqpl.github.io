CVE-2023-26465 - Breaking Through XSS Filters with Markdown-nesting and User Mentioning in Pega Platform

Last year we identified an interesting XSS vulnerability involving clever use of markdown syntaxt and user mentioning in Pega Systems Platform. This post delves into details of the PoC, providing a concise yet thorough analysis of how arbitrary JavaScript code could be executed within the application.

# Intro

Pega Platform is a complex CRM solution designed to automate business processes and improve customer engagement. It includes many sub-modules, one of which is the Pega Pulse, facilitating direct communication and collaboration within the platform. 

Pega Pulse allows for use of markdown syntax as well as mention other users in the company, we exploited those two functionalities to construct the below PoC.

# PoC


```
![@maciej.piechota@secforce.com <![img src=x onerror=alert(window.origin)](1)>](1)
```

# PoC Deconstruction

In the beginning, we noticed that it is possible to escape from quotes in anchor tags using user-mentioning functionality:

```html
<a href="https://secforce.com@maciej.piechota@secforce.com ">link</a>
```

{% include figure.html file="/assets/6d0852c81194447887189384bf4b2ee5.png" alt="/assets/6d0852c81194447887189384bf4b2ee5.png" max-width="500px" number="6" caption="PoC" %}

please notice that part of the mention became the HTML attribute of the anchor tag.

We tried to construct a payload using any controlled part of HTML produced by mention but without results.

Then we focused on other functionalities and quickly noticed that limited markdown syntax is usable, among other images and links:

```
![secforce](1)
```

produces:

{% include figure.html file="/assets/a155f1e233974fa9a4e4481ea24bf3cb.png" alt="/assets/a155f1e233974fa9a4e4481ea24bf3cb.png" max-width="500px" number="6" caption="MD Image" %}

with controlled alternative text attribute, so we thought we could construct a payload there and then escape quotes using the previous method.

Unfortunately, HTML tags were stripped from the `alt` attribute, however, we did the same trick with nesting to bypass the XSS filter:

```
![<![img src=x onerror=alert(window.origin)](1)>](1)
```

what allowed us to construct an XSS payload inside the `alt` attribute.

{% include figure.html file="/assets/b1f35799267c4fd28806100877b8a7de.png" alt="/assets/b1f35799267c4fd28806100877b8a7de.png" max-width="500px" number="6" caption="alt attribute" %}

Utilizing the earlier method of escaping from an attribute resulted in a stored cross-site scripting vulnerability.

```
![@maciej.piechota@secforce.com <![img src=x onerror=alert(window.origin)](1)>](1)
```
{% include figure.html file="/assets/d58fdc0bf9634d368d5c7c87359045d8.png" alt="/assets/d58fdc0bf9634d368d5c7c87359045d8.png" max-width="500px" number="6" caption="attribute escape" %}

{% include figure.html file="/assets/3dcc89d8dd6443f3a4e5ac2bc4b29d0b.png" alt="/assets/3dcc89d8dd6443f3a4e5ac2bc4b29d0b.png" max-width="500px" number="6" caption="alert fired" %}


# Affected versions 

Pega Platform <= 8.7.1


# Timeline
<pre>
• 08.04.2022 – reported XSS to Pega Security team
• 31.05.2022 – team sent acknowledgement
• 24.02.2023 – CVE-2023-26465 assigned
• 30.05.2023 – Pega Security Advisory released 
</pre>

# References

https://support.pega.com/support-doc/pega-security-advisory-a23-vulnerability-remediation-note
https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2023-26465

# Outro 

I would like to thank Pega security team for a smooth colaboration during responsible disclosure of this vulnerability.
