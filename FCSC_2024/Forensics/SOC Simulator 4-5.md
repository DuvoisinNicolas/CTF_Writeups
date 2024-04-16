Login failure for "admin_bidon" ahah:

![](_attachments/Pasted%20image%2020240407102231.png)

I also found this, filtering by event id "4625" (Logon success or failure):

![](_attachments/Pasted%20image%2020240407102421.png)

All computers tried to log in...
Within a second ?
Looks like password spraying!

![](_attachments/Pasted%20image%2020240407102801.png)
![](_attachments/Pasted%20image%2020240407102949.png)

Here we can see a session of Administrator opening on Workstation2.
Let's try this for the flag ?
`FCSC{172.16.20.20|WORKSTATION2\Administrator|2022-07-06T13:26}`
Nah.
Maybe that's the domain admin ? TINFA\Administrator ?
Didn't work either:
`FCSC{172.16.20.20|Administrator|2022-07-06T13:26}`
`FCSC{172.16.20.20|TINFA\Administrator|2022-07-06T13:26}`

Ip source looks good, and Timestamp is 99%.
So the issue is probably the account...

After searching, i also found:

![](_attachments/Pasted%20image%2020240407103629.png)

So that might be here first.

I tried with "stephanie.amasse" because she logged in in that time period, but that's probably a false positive.
`FCSC{172.16.10.201|TINFA\stephanie.amasse|2022-07-06T13:26}`

![](_attachments/Pasted%20image%2020240407110633.png)

I also found this event... Let's try

`FCSC{172.16.20.20|TINFA\ben.bidon|2022-07-06T14:34}`

Oh gosh, they want the seconds !!!!
`FCSC{172.16.20.20|Workstation2\Administrator|2022-07-06T13:26:57}`
Nevermind, this flag worked ! :)