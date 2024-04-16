So here, we're told that they removed /bin/sh from the container, so we can't use "system" "/bin/sh" in order to get a shell.
But i think we could use "system" "/bin/ls" in order to list the folder, and then use "read" or something to get the flag once we have the full path.

After reading, i found that "system" already runs "sh -c XXX", so that won't work.
I looked for other shells:
```
cat /etc/shells
# /etc/shells: valid login shells
/bin/sh
/usr/bin/sh
/bin/bash
/usr/bin/bash
/bin/rbash
/usr/bin/rbash
/bin/dash
/usr/bin/dash
```