Now we have to look for dump of credentials from memory.

![](_attachments/Pasted%20image%2020240406225218.png)

I found this event searching for "comsvcs", an usual dll used to dump LSASS Memory.
The previous event had these data:

![](_attachments/Pasted%20image%2020240406225551.png)

Using these informations, the flag is simple to guess:

`FCSC{b99a131f-0d4b-62c3-ce03-00000000db01|C:\Windows\System32\inetsrv\attr.exe}`

