So, this one's talking about CTI, nice !

So we need to identify the attacker group behind that Ip, and also the service running on the IP.

We found in part 2 that the IP was: `20.13.3.0`.

Zero ? In an IP ? :o

Didn't know that was possible...

Let's google this:

![](_attachments/Pasted%20image%2020240409185053.png)

Looks like our friends from Netherlands ! (or a VPN)

I'm going to use this website:
https://hackcontrol.org/OSINT/IP_addres.html

It has many (too much?) resources, including Ip address processing.

On Shodan, we can see this:
https://www.shodan.io/host/20.13.3.0

![](_attachments/Pasted%20image%2020240409185508.png)
![](_attachments/Pasted%20image%2020240409185515.png)

Well not that surprising.

The timestamps of our logs is:
`Fri Mar 15 06:27:49 2024`

And the website says:

![](_attachments/Pasted%20image%2020240409185631.png)

Definitely the right ip.

Now we just need to find the attacker group name.

I found:
FCSC{UNC5221:OpenSSH}
But they didn't work.

I found this twitter account

![](_attachments/Pasted%20image%2020240409191545.png)

Talking about Ivanti and UNC5521, i'm trying this one.

Didn't work either...

Only 4 tries remaining !

https://thehackernews.com/2024/03/ivanti-releases-urgent-fix-for-critical.html?fbclid=IwAR3CxbuPjWndPCkYoTRL9B03yduCrWOKRAc93GJiyPOq1o6QpnG6NgMtgNk_aem_ASlYVpaC-mKEy1pXvC6YIo3XlYsuxrzhZSODQIw7SHIRi3zF1z8cd9OnyXHtbmOKrzVVCoVnyU_8GjN2QnNePguv

Here they're talking about  [UNC5221](https://thehackernews.com/2024/02/warning-new-malware-emerges-in-attacks.html), [UNC5325, and UNC3886](https://thehackernews.com/2024/02/chinese-hackers-exploiting-ivanti-vpn.html), but i'm scared now ahah

https://thehackernews.com/2024/02/warning-new-malware-emerges-in-attacks.html

![](_attachments/Pasted%20image%2020240409192359.png)

This pages talks about the group, and the CVE...


`FCSC{UNC5221:OpenSSH}` didn't work either...

`FCSC{UNC5221:ssh}`.

Nevermind, i'm dumb ! :)

On the 3rd part, we saw this command:
```bash
nc 146.0.228.66:1337
```

This is probably the IP we're supposed to scan, so the service isn't SSH.

![](_attachments/Pasted%20image%2020240409194414.png)

I found many reports talking about C2 warpwire.

I tried warp wire...
```
FCSC{UNC5221:Warpwire}
```

And this failed :(

I only have 1 try left.

https://1275.ru/ioc/3035/unc5221-apt-iocs-part-2/
This pages confirms that the attacker group is UNC5221.

![](_attachments/Pasted%20image%2020240409195304.png)


Gosh what this service could be...

On a random google page, i found this:

![](_attachments/Pasted%20image%2020240409195547.png)

It says that the port RDP was open.
The challenge talks about "the legitimate management interface".
That looks good ?

But i've only 1 try left :(

![](_attachments/Pasted%20image%2020240409195916.png)

This one talks about 1080, but it's for a SOCKS server, not sure that's a "management interface".


I found this:

![](_attachments/Pasted%20image%2020240409213851.png)

That says that Areekaweb.com was the domain name for the server from May 2022 to the 09th march 2024.

I did a research on that website:
https://centralops.net/co/DomainDossier.aspx

Of the domain name: areekaweb.com

And i saw this in the HTTP headers:
```
HTTP/1.1 200 OK 
Cache-Control: private 
Content-Length: 56452 
Content-Type: text/html; charset=utf-8 
Server: Microsoft-IIS/10.0 
X-AspNetMvc-Version: 5.2 
X-Frame-Options: SAMEORIGIN 
Set-Cookie: AREEKALang=En-US; expires=Wed, 09-Apr-2025 20:59:08 GMT; path=/; HttpOnly 
Set-Cookie: __RequestVerificationToken=A1NkuTYKFe03F76b3EQR2zQcnieRBBKf1QpYiifj3SgepGBH3H1zXX9YtQBiyapAUoGHqmy55ZlHJf7Tq6oEFQo3uRKrtyc0hPKBHYDsKpw1; path=/; HttpOnly 
X-Powered-By: ASP.NET 
X-Powered-By-Plesk: PleskWin Date: Tue, 09 Apr 2024 20:59:08 GMT Connection: close
```

And then i saw:
```
X-Powered-By-Plesk:
```

I googled...

![](_attachments/Pasted%20image%2020240409230413.png)

In a last resort, i tried this flag:
```
FCSC{UNC5221:plesk}
```
And that WORKED ! :3

Close one !