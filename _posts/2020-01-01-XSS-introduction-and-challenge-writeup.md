---
title: XSS introduction and challenge writeup.
published: false
author: Maciej Piechota
---

A few months ago I took place in the XSS challenge organized by @haxel0rd and later was asked about explaining my solutions. The challenge was divided into levels starting from easy to the hard one, each level was about exploiting different XSS context, which was great in terms of learning sake. I will describe each solution and some theory behind them, so let's dive in.

## The challenge:

<div><div class="css-1dbjc4n r-1ila09b r-qklmqi r-1adg3ll"><article aria-haspopup="false" role="article" data-focusable="true" tabindex="0" class="css-1dbjc4n r-1loqt21 r-1udh08x"><div class="css-1dbjc4n r-1j3t67a"><div class="css-1dbjc4n r-18u37iz r-thb0q2"><div class="css-1dbjc4n r-1iusvr4 r-16y2uox r-5f2r5o r-m611by"></div></div><div class="css-1dbjc4n r-18u37iz r-thb0q2 r-1mi0q7o" data-testid="tweet"><div class="css-1dbjc4n r-1awozwy r-18kxxzh r-5f2r5o" style="flex-basis: 49px;"><div class="css-1dbjc4n r-18kxxzh r-1wbh5a2 r-13qz1uu"><div class="css-1dbjc4n r-1wbh5a2 r-dnmrzs"><a aria-haspopup="false" href="/haxel0rd" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-1dbjc4n r-sdzlij r-1loqt21 r-1adg3ll r-ahm1il r-1udh08x r-o7ynqc r-6416eg r-13qz1uu"><div class="css-1dbjc4n r-1adg3ll r-1udh08x" style=""><div class="r-1adg3ll r-13qz1uu" style="padding-bottom: 100%;"></div><div class="r-1p0dtai r-1pi2tsx r-1d2f490 r-u8s1d r-ipm5af r-13qz1uu"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-1mlwlqe r-1d2f490 r-1udh08x r-u8s1d r-zchlnj r-ipm5af r-417010"><div class="css-1dbjc4n r-1niwhzg r-vvn4in r-u6sd8q r-4gszlv r-1p0dtai r-1pi2tsx r-1d2f490 r-u8s1d r-zchlnj r-ipm5af r-13qz1uu r-1wyyakw" style="background-image: url(&quot;https://pbs.twimg.com/profile_images/700425058821525506/9OON01Oa_bigger.jpg&quot;);"></div><img alt="" draggable="true" src="https://pbs.twimg.com/profile_images/700425058821525506/9OON01Oa_bigger.jpg" class="css-9pa8cd"></div></div></div><div aria-haspopup="false" class="css-1dbjc4n r-1twgtwe r-sdzlij r-rs99b7 r-1p0dtai r-1mi75qu r-1d2f490 r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div></a></div></div></div><div class="css-1dbjc4n r-1iusvr4 r-16y2uox r-1777fci r-5f2r5o"><div class="css-1dbjc4n r-18u37iz r-1wtj0ep r-zl2h9q"><div class="css-1dbjc4n r-1wbh5a2 r-dnmrzs"><a aria-haspopup="false" href="/haxel0rd" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-1dbjc4n r-1loqt21 r-1wbh5a2 r-dnmrzs r-1ny4l3l"><div class="css-1dbjc4n r-1wbh5a2 r-dnmrzs r-1ny4l3l"><div class="css-1dbjc4n r-18u37iz r-dnmrzs"><div dir="auto" class="css-901oao css-bfa6kz r-jwli3a r-1qd0xha r-a023e6 r-vw2c0b r-ad9z0x r-bcqeeo r-3s2u2q r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">Haxel0rd</span></span></div><div dir="auto" class="css-901oao r-jwli3a r-18u37iz r-1q142lx r-1qd0xha r-a023e6 r-16dba41 r-ad9z0x r-bcqeeo r-qvutc0"></div></div><div class="css-1dbjc4n r-18u37iz r-1wbh5a2"><div dir="ltr" class="css-901oao css-bfa6kz r-111h2gw r-18u37iz r-1qd0xha r-a023e6 r-16dba41 r-ad9z0x r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">@haxel0rd</span></div></div></div></a></div><div class="css-1dbjc4n r-k200y r-18u37iz r-1h0z5md r-1joea0r"><div aria-haspopup="true" aria-label="Więcej" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-1dbjc4n r-1777fci r-11cpok1 r-1ny4l3l r-bztko3 r-lrvibr" data-testid="caret"><div dir="ltr" class="css-901oao r-1awozwy r-111h2gw r-6koalj r-1qd0xha r-a023e6 r-16dba41 r-1h0z5md r-ad9z0x r-bcqeeo r-o7ynqc r-clp7b1 r-3s2u2q r-qvutc0"><div class="css-1dbjc4n r-xoduu5"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-xoduu5 r-1d2f490 r-podbf7 r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div><svg viewBox="0 0 24 24" class="r-4qtqp9 r-yyyyoo r-ip8ujx r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-27tl0q"><g><path d="M20.207 8.147c-.39-.39-1.023-.39-1.414 0L12 14.94 5.207 8.147c-.39-.39-1.023-.39-1.414 0-.39.39-.39 1.023 0 1.414l7.5 7.5c.195.196.45.294.707.294s.512-.098.707-.293l7.5-7.5c.39-.39.39-1.022 0-1.413z"></path></g></svg></div></div></div></div></div></div></div><div class="css-1dbjc4n"><div dir="auto" class="css-901oao r-jwli3a r-1qd0xha r-1blvdjr r-16dba41 r-ad9z0x r-bcqeeo r-19yat4t r-bnwqim r-qvutc0" lang="en"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">Can you beat all 10 levels..?
</span><span class="r-18u37iz"><a href="/hashtag/XSS?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#XSS</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><span class="r-18u37iz"><a href="/hashtag/WAF?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#WAF</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><span class="r-18u37iz"><a href="/hashtag/FilterEvasion?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#FilterEvasion</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><span class="r-18u37iz"><a href="/hashtag/Challenge?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#Challenge</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">, from Beginner =&gt; Hard, pure XSS, no fantasy vectors. Will post first 7 solvers on my twitter. The challenge starts at login (no-xss), good luck. </span><span class="r-18u37iz"><a href="/hashtag/Hacking?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#Hacking</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><span class="r-18u37iz"><a href="/hashtag/HackIt?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#HackIt</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><span class="r-18u37iz"><a href="/hashtag/CTF?src=hashtag_click" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">#CTF</a></span><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> </span><a title="https://u.nu/hackit" href="https://t.co/6Vwu5Z8NEy?amp=1" target="_blank" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0" rel=" noopener noreferrer"><span aria-hidden="true" class="css-901oao css-16my406 r-1qd0xha r-hiw28u r-ad9z0x r-bcqeeo r-qvutc0">https://</span>u.nu/hackit</a><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"> CodedBy: </span><div class="css-1dbjc4n r-xoduu5"><span class="r-18u37iz"><a href="/ObscurityApp" dir="ltr" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">@ObscurityApp</a></span></div></div><div aria-expanded="false" dir="auto" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-901oao r-1n1174f r-6koalj r-1w6e6rj r-1qd0xha r-n6v787 r-16dba41 r-1sf4r6n r-1g94qm0 r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">Przetłumacz Tweeta</span></div></div><div class="css-1dbjc4n r-1awozwy r-18u37iz r-1wtj0ep r-ku1wi2"><div dir="auto" class="css-901oao r-111h2gw r-1qd0xha r-a023e6 r-16dba41 r-ad9z0x r-zso239 r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-111h2gw r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">11:58 PM · 12 kwi 2019</span></span><span aria-hidden="true" class="css-901oao css-16my406 r-111h2gw r-1q142lx r-1qd0xha r-ad9z0x r-bcqeeo r-ou255f r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">·</span></span><a href="https://help.twitter.com/using-twitter/how-to-tweet#source-labels" target="_blank" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao css-16my406 r-1n1174f r-1loqt21 r-1qd0xha r-ad9z0x r-bcqeeo r-1jeg54m r-qvutc0" rel=" noopener noreferrer"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">Twitter Web Client</span></a></div></div><div class="css-1dbjc4n r-1kfrmmb r-1efd50x r-5kkj8d r-18u37iz r-9qu9m4"><div class="css-1dbjc4n"><a href="/haxel0rd/status/1116822993085325312/retweets" dir="auto" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao r-jwli3a r-1loqt21 r-1qd0xha r-a023e6 r-16dba41 r-ad9z0x r-bcqeeo r-qvutc0"><div class="css-1dbjc4n r-xoduu5 r-1udh08x"><span class="css-901oao css-16my406 r-1qd0xha r-vw2c0b r-ad9z0x r-bcqeeo r-d3hbe1 r-1wgg2b2 r-axxi2z r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">28</span></span></div>
          <span class="css-901oao css-16my406 r-111h2gw r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">podań dalej</span></span></a></div><div class="css-1dbjc4n r-1joea0r"><a href="/haxel0rd/status/1116822993085325312/likes" dir="auto" role="link" data-focusable="true" class="css-4rbku5 css-18t94o4 css-901oao r-jwli3a r-1loqt21 r-1qd0xha r-a023e6 r-16dba41 r-ad9z0x r-bcqeeo r-qvutc0"><div class="css-1dbjc4n r-xoduu5 r-1udh08x"><span class="css-901oao css-16my406 r-1qd0xha r-vw2c0b r-ad9z0x r-bcqeeo r-d3hbe1 r-1wgg2b2 r-axxi2z r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">60</span></span></div>
            <span class="css-901oao css-16my406 r-111h2gw r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0"><span class="css-901oao css-16my406 r-1qd0xha r-ad9z0x r-bcqeeo r-qvutc0">Polubień</span></span></a></div></div><div aria-label="5 odpowiedzi, 28 Tweetów podanych dalej, 60 polubień, Lubi" role="group" class="css-1dbjc4n r-1oszu61 r-1kfrmmb r-1efd50x r-5kkj8d r-18u37iz r-ahm1il r-a2tzq0"><div class="css-1dbjc4n r-18u37iz r-1h0z5md r-3qxfft r-h4g966 r-rjfia"><div aria-haspopup="false" aria-label="Odpowiedz" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-1dbjc4n r-1777fci r-11cpok1 r-1ny4l3l r-bztko3 r-lrvibr" data-testid="reply"><div dir="ltr" class="css-901oao r-1awozwy r-111h2gw r-6koalj r-1qd0xha r-a023e6 r-16dba41 r-1h0z5md r-ad9z0x r-bcqeeo r-o7ynqc r-clp7b1 r-3s2u2q r-qvutc0"><div class="css-1dbjc4n r-xoduu5"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-xoduu5 r-1d2f490 r-xf4iuw r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div><svg viewBox="0 0 24 24" class="r-4qtqp9 r-yyyyoo r-50lct3 r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-1srniue"><g><path d="M14.046 2.242l-4.148-.01h-.002c-4.374 0-7.8 3.427-7.8 7.802 0 4.098 3.186 7.206 7.465 7.37v3.828c0 .108.044.286.12.403.142.225.384.347.632.347.138 0 .277-.038.402-.118.264-.168 6.473-4.14 8.088-5.506 1.902-1.61 3.04-3.97 3.043-6.312v-.017c-.006-4.367-3.43-7.787-7.8-7.788zm3.787 12.972c-1.134.96-4.862 3.405-6.772 4.643V16.67c0-.414-.335-.75-.75-.75h-.396c-3.66 0-6.318-2.476-6.318-5.886 0-3.534 2.768-6.302 6.3-6.302l4.147.01h.002c3.532 0 6.3 2.766 6.302 6.296-.003 1.91-.942 3.844-2.514 5.176z"></path></g></svg></div></div></div></div><div class="css-1dbjc4n r-18u37iz r-1h0z5md r-3qxfft r-h4g966 r-rjfia"><div aria-haspopup="true" aria-label="Podaj dalej" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-1dbjc4n r-1777fci r-11cpok1 r-1ny4l3l r-bztko3 r-lrvibr" data-testid="retweet"><div dir="ltr" class="css-901oao r-1awozwy r-111h2gw r-6koalj r-1qd0xha r-a023e6 r-16dba41 r-1h0z5md r-ad9z0x r-bcqeeo r-o7ynqc r-clp7b1 r-3s2u2q r-qvutc0"><div class="css-1dbjc4n r-xoduu5"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-xoduu5 r-1d2f490 r-xf4iuw r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div><svg viewBox="0 0 24 24" class="r-4qtqp9 r-yyyyoo r-50lct3 r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-1srniue"><g><path d="M23.77 15.67c-.292-.293-.767-.293-1.06 0l-2.22 2.22V7.65c0-2.068-1.683-3.75-3.75-3.75h-5.85c-.414 0-.75.336-.75.75s.336.75.75.75h5.85c1.24 0 2.25 1.01 2.25 2.25v10.24l-2.22-2.22c-.293-.293-.768-.293-1.06 0s-.294.768 0 1.06l3.5 3.5c.145.147.337.22.53.22s.383-.072.53-.22l3.5-3.5c.294-.292.294-.767 0-1.06zm-10.66 3.28H7.26c-1.24 0-2.25-1.01-2.25-2.25V6.46l2.22 2.22c.148.147.34.22.532.22s.384-.073.53-.22c.293-.293.293-.768 0-1.06l-3.5-3.5c-.293-.294-.768-.294-1.06 0l-3.5 3.5c-.294.292-.294.767 0 1.06s.767.293 1.06 0l2.22-2.22V16.7c0 2.068 1.683 3.75 3.75 3.75h5.85c.414 0 .75-.336.75-.75s-.337-.75-.75-.75z"></path></g></svg></div></div></div></div><div class="css-1dbjc4n r-18u37iz r-1h0z5md r-3qxfft r-h4g966 r-rjfia"><div aria-haspopup="false" aria-label="Lubi" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-1dbjc4n r-1777fci r-11cpok1 r-1ny4l3l r-bztko3 r-lrvibr" data-testid="unlike"><div dir="ltr" class="css-901oao r-1awozwy r-daml9f r-6koalj r-1qd0xha r-a023e6 r-16dba41 r-1h0z5md r-ad9z0x r-bcqeeo r-o7ynqc r-clp7b1 r-3s2u2q r-qvutc0"><div class="css-1dbjc4n r-xoduu5"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-xoduu5 r-1d2f490 r-xf4iuw r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div><svg viewBox="0 0 24 24" class="r-4qtqp9 r-yyyyoo r-50lct3 r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-1srniue"><g><path d="M12 21.638h-.014C9.403 21.59 1.95 14.856 1.95 8.478c0-3.064 2.525-5.754 5.403-5.754 2.29 0 3.83 1.58 4.646 2.73.814-1.148 2.354-2.73 4.645-2.73 2.88 0 5.404 2.69 5.404 5.755 0 6.376-7.454 13.11-10.037 13.157H12z"></path></g></svg></div></div></div></div><div class="css-1dbjc4n r-18u37iz r-1h0z5md r-3qxfft r-h4g966 r-rjfia"><div aria-haspopup="true" aria-label="Udostępnij Tweeta" role="button" data-focusable="true" tabindex="0" class="css-18t94o4 css-1dbjc4n r-1777fci r-11cpok1 r-1ny4l3l r-bztko3 r-lrvibr"><div dir="ltr" class="css-901oao r-1awozwy r-111h2gw r-6koalj r-1qd0xha r-a023e6 r-16dba41 r-1h0z5md r-ad9z0x r-bcqeeo r-o7ynqc r-clp7b1 r-3s2u2q r-qvutc0"><div class="css-1dbjc4n r-xoduu5"><div class="css-1dbjc4n r-sdzlij r-1p0dtai r-xoduu5 r-1d2f490 r-xf4iuw r-u8s1d r-zchlnj r-ipm5af r-o7ynqc r-6416eg"></div><svg viewBox="0 0 24 24" class="r-4qtqp9 r-yyyyoo r-50lct3 r-dnmrzs r-bnwqim r-1plcrui r-lrvibr r-1srniue"><g><path d="M17.53 7.47l-5-5c-.293-.293-.768-.293-1.06 0l-5 5c-.294.293-.294.768 0 1.06s.767.294 1.06 0l3.72-3.72V15c0 .414.336.75.75.75s.75-.336.75-.75V4.81l3.72 3.72c.146.147.338.22.53.22s.384-.072.53-.22c.293-.293.293-.767 0-1.06z"></path><path d="M19.708 21.944H4.292C3.028 21.944 2 20.916 2 19.652V14c0-.414.336-.75.75-.75s.75.336.75.75v5.652c0 .437.355.792.792.792h15.416c.437 0 .792-.355.792-.792V14c0-.414.336-.75.75-.75s.75.336.75.75v5.652c0 1.264-1.028 2.292-2.292 2.292z"></path></g></svg></div></div></div></div></div></div></article></div></div>

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

{% include figure.html file="/assets/lvl3_1.png" alt="/assets/lvl3_1.png" max-width="500px" number="1" caption="Level 03 - Payload." %}

## Lvl04:

This level was solved with unintended solution and introduces to us the next injection context - JavaScript context.

{% include figure.html file="/assets/lvl4.png" alt="/assets/lvl4.png" max-width="500px" number="1" caption="Level 04 - JavaScript context." %}

Developer console shows the JavaScript error: 

{% include figure.html file="/assets/lvl4_1.png" alt="/assets/lvl4_1.png" max-width="500px" number="1" caption="Level 04 - JavaScript error." %}

which tells us that execution of the JavaScript was interupted because of our injected payload. The steps to do in that case are usually the same:

1. Close all opened strings, parentheses, remember that the JavaScript before injection must be "satisfied".
2. Add `;` as we want to start new instruction.
3. alert(1)
4. Deal with code which was there before injection making it as a comment with `//`.

Solution:
`xsshere'); alert(1)//`

Intended solution:

{% include figure.html file="/assets/lvl4_2.png" alt="/assets/lvl4_2.png" max-width="500px" number="1" caption="Level 04 - JavaScript error." %}

## Lvl05:

{% include figure.html file="/assets/lvl5.png" alt="/assets/lvl5.png" max-width="500px" number="1" caption="Level 05 - JavaScript variable context." %}

This is very similar to the previous level.

We're in the JavaScript context.

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

but there is a filter, we can't start new `script` tag. JavaScript implements three characters which identify strings: `'`, `"` and `&grave;`. So payload like: `xsshere&grave;; alert(1)//` is solution for this lvl.

## TBC

## Credits:

@haxel0rd
