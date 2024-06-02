---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: todo-hard
ctf-category: web
ctf-difficulty: 
---
# todo-hard

## Description

> I made a JS API! Sadly I had no time to finish it :(
> 
> But I had time to make it harder!

## Challenge

This is similar to the `todo` challenge, but this time, the flag in the response will be replaced by `"nope"`:
```js
app.post('/admin', async (req, res) => {
    try {
        const { html } = req.body;
        const browser = await puppeteer.launch({ executablePath: process.env.BROWSER, args: ['--no-sandbox'] }); 
        const page = await browser.newPage();
        page.setCookie({ name: 'flag', value: randomBytes, domain: 'localhost', path: '/', httpOnly: true }); 
        await page.goto('http://localhost:1337/');
        await page.type('input[name="html"]', html);
        await page.click('button[type="submit"]');
        await new Promise(resolve => setTimeout(resolve, 2000));
        // NEW: run JS to replace the flag with "nope"
        await page.evaluate((flag) => { document.body.outerHTML = document.body.outerHTML.replace(flag, "nope") },     flag)
        const screenshot = await page.screenshot({ encoding: 'base64' }); 
        await browser.close();
        res.send(`<img src="data:image/png;base64,${screenshot}" />`);
    } catch(e) {console.error(e); res.send("internal error :( pls report to admins")}
});
```

## Solution

The prototype of the `replace` function can be overwritten, so that another "replace" function is executed which just returns the first argument, instead of performing an actual replacement:
```html
<script>
function Fun(a, b){ return a;};
String.prototype.replace = Fun;
</script>
```

The flag can then be seen in the screenshot in the response:
![](_attachments/Pasted%20image%2020240601215138.png)