Here i've to find stolen data that has been zipped or some kind.

![](_attachments/Pasted%20image%2020240406233415.png)

That one, using powershell in the folder where we found the dumped credentials...

![](_attachments/Pasted%20image%2020240406233852.png)

Dlhost running a process ?

![](_attachments/Pasted%20image%2020240406234814.png)

Also here we can see him echoing "appdata", so maybe he went there to do something.


Mhh not many things right now...
This one has been flagged only 2 times at the time of writing, when others have 15+.
So it's probably a harder one.
Let's enumerate the things that has been done on the computer "exchange", since this is where the attacker is supposed to be.
Nothing much interesting.

I tried to look for request from our IP to "fileserver", maybe that could give us a nudge of when the user tried to steal data.

![](_attachments/Pasted%20image%2020240407143621.png)


I found that if we look for "aspx" files in logs, we can identify when the attacker reconnected his reverse shell.

![](_attachments/Pasted%20image%2020240407211522.png)

I'm doing this challenge last (already did other parts), so i filtered out event outside of between part 2 and part 4 timestamps.

So, here are our possible timestamps:
- 05/07 10:49:57
- 05/07 15:21:19
- 05/07 17:41:10
- 05/07 17:58:10
- 06/07 11:01:16

The last one is probably before doing the malicious things of part 4.

So, let's check in windows events if we see something around these timestamps.

- 05/07 10:49:57: Didn't see much.
- 05/07 15:21:19:

![](_attachments/Pasted%20image%2020240407214129.png)

I found some other powershell scripts around that time.
I'll try to decode them too, like i did in step 1.

We can see the command using "Remove-MailboxExportRequest"

![](_attachments/Pasted%20image%2020240407215225.png)

- 05/07 17:41:10:
- Nothing interesting


- 05/07 17:58:10:
I found that the user "serge.huon" is running weird python things, accessing to Finance fileserver... and he event ran echo local app data to the cmd...

![](_attachments/Pasted%20image%2020240407223228.png)

Later i found:

![](_attachments/Pasted%20image%2020240407223831.png)

stephanie.amasse that access  to Management file server...

That might be a "normal" behaviour.

- 06/07 11:01:16