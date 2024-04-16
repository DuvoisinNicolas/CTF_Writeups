So, we know our attacker went on Workstation2 as administrator.

So i filtered the logs that happenned on Workstation2 after his loggon.

![](_attachments/Pasted%20image%2020240407112038.png)

There are 2 logs:

![](_attachments/Pasted%20image%2020240407112054.png)

ben.bidon, running lsass.exe 1 hour after our admin logged in

![](_attachments/Pasted%20image%2020240407112124.png)

And the administrator, using internet explorer to dump credentials in memory.

Now we just need to find the GUID of iexplore.exe...
`FCSC{XXX|C:\Users\Administrator\AppData\Local\Temp\3\lsass.DMP}`

I tried some GUID i found

- 133EAC4F-5891-4D04-BADA-D84870380A80
- 9aa46009-3ce0-458a-a354-715610a075e6
- b7e8a6b7-b23e-62c5-a324-df0500000000
- AB8902B4-09CA-4BB6-B78D-A8F59079A8D5
- b7e8a6b7-b245-62c5-9a11-00000000d301
- b7e8a6b7-b2e2-62c5-e711-00000000d301
- b7e8a6b7-b2e2-62c5-e811-00000000d301
- b7e8a6b7-b2e8-62c5-e911-00000000d301
- b7e8a6b7-b2e9-62c5-ea11-00000000d301
- b7e8a6b7-b2ee-62c5-eb11-00000000d301
But none worked...


After many researches, i found this log:

![](_attachments/Pasted%20image%2020240407132500.png)

`FCSC{b7e8a6b7-b273-62c5-bc11-00000000d301|C:\Users\Administrator\AppData\Local\Temp\3\lsass.DMP}`
And that one worked ! :)

