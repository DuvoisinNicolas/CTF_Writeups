```
Un nouveau ransomware se propage sur internet.

Trop de vieilles dames se font arnaquer par celui-ci, il est temps d'agir !

Une des victimes nous a accordé l'accès à distance à sa machine, veuillez enquêter et trouver la clé pour déchiffrer les fichiers.

Attention à ne pas lancer le ransomware sur une machine autre que celle fournie.
```

I reversed the file a bit, and saw it was doing some symetric encryption using Register keys.

I went on the distant instance, and used ProcMon to show the Registers accesses from wrongsomewhere.exe.

I found that the process was using a specific register key hidden in Onedrive data, named "error".

I used this value as a key to decrypt files, and found the flag !

`DGHACK{R4nS0mW4r3s_4r3_4_Cr1m3_D0_n0t_Us3_Th1s_0N3_F0r_3v1l}`
