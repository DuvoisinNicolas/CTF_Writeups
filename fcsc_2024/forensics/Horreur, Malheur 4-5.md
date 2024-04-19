So now we're looking for a procotol, a port, an absolute path for an edited configuration file, and an absolute path for another persistance file.

Also, we only have 10 attempts for this flag.

First, i'm going to look at a protocol list, as this could allow us to find a foothold.

## **Types of Internet Protocol**

Internet Protocols are of different types having different uses. These are mentioned below:

1. [TCP/IP(Transmission Control Protocol/ Internet Protocol)](https://www.geeksforgeeks.org/tcp-ip-model/)
2. [SMTP(Simple Mail Transfer Protocol)](https://www.geeksforgeeks.org/simple-mail-transfer-protocol-smtp/)
3. [PPP(Point-to-Point Protocol)](https://www.geeksforgeeks.org/point-to-point-protocol-ppp-frame-format/)
4. [FTP (File Transfer Protocol)](https://www.geeksforgeeks.org/file-transfer-protocol-ftp-in-application-layer/)
5. [SFTP(Secure File Transfer Protocol)](https://www.geeksforgeeks.org/sftp-file-transfer-protocol/)
6. [HTTP(Hyper Text Transfer Protocol)](https://www.geeksforgeeks.org/http-full-form/)
7. [HTTPS(HyperText Transfer Protocol Secure)](https://www.geeksforgeeks.org/explain-working-of-https/)
8. [TELNET(Terminal Network)](https://www.geeksforgeeks.org/introduction-to-telnet/)
9. [POP3(Post Office Protocol 3)](https://www.geeksforgeeks.org/pop-full-form/)
10. [IPv4](https://www.geeksforgeeks.org/what-is-ipv4/)
11. [IPv6](https://www.geeksforgeeks.org/what-is-ipv6/)
12. [ICMP](https://www.geeksforgeeks.org/internet-control-message-protocol-icmp/)
13. [UDP](https://www.geeksforgeeks.org/user-datagram-protocol-udp/)
14. [IMAP](https://www.geeksforgeeks.org/internet-message-access-protocol-imap/)
15. [SSH](https://www.geeksforgeeks.org/introduction-to-sshsecure-shell-keys/)
16. Gopher

Let's grep some of these ones in the logs.

Also, on the last part, we found this:
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

Which seems to edit the ssh config file.

Let's search for SSH, and ports 1337 !

We can find this line on our logs:
```bash
sshd-ive  22500     root    3u     IPv4            1585502       0t0        TCP 172.18.0.4:1337 (LISTEN)
```

So, we already know most of the things if that's the right thing to look for.

```
Protocol: SSH
Port: 1337 (Leet :3)
Edited configuration file: "/data/runtime/etc/ssh/sshd_server_config"
Persistance file: "/data/runtime/etc/ssh/ssh_host_rsa_key.pub"
```

Can be that easy... Could it be ?
`FCSC{ssh:1337:/data/runtime/etc/ssh/sshd_server_config:/data/runtime/etc/ssh/ssh_host_rsa_key.pub}`

Nah.

Might try SSH in uppercase, just in... case :')

Nah.
```
FCSC{ssh:1337:/data/runtime/etc/ssh/sshd_server_config:/data/runtime/etc/ssh/ssh_host_rsa_key.pub}
```
Not that either.

Let's comment the above commands, i probably missed something:
```bash
# Replace the default port with 1337 (netconf-ssh default port was 830)
sed -i 's/port 830/port 1337/' 
/data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1

# Comment the "force command" parameter
sed -i 's/ForceCommand/#ForceCommand/' /data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1

# Allow pubkey authentication
echo "PubkeyAuthentication yes" >> /data/runtime/etc/ssh/sshd_server_config

# Authorize the pubkey "ssh_host_rsa_key.pub"
echo "AuthorizedKeysFile /data/runtime/etc/ssh/ssh_host_rsa_key.pub" >> /data/runtime/etc/ssh/sshd_server_config

# Stop sshd-ive
pkill sshd-ive > /dev/null 2>&1

# Unzip the config
gzip -d /data/pkg/data-backup.tgz > /dev/null 2>&1

# Tar the sshd config
tar -rf /data/pkg/data-backup.tar /data/runtime/etc/ssh/sshd_server_config > /dev/null 2>&1

# Tar gz the sshd config
gzip /data/pkg/data-backup.tar > /dev/null 2>&1

# Rename it
mv /data/pkg/data-backup.tar.gz /data/pkg/data-backup.tgz > /dev/null 2>&1
```

He also replaced the old data-backup with a new one, with the edited sshd config.

Maybe that's the file they're talking about for "the other persistance file" ?

Let's try !

Flagged :)
```
FCSC{ssh:1337:/data/runtime/etc/ssh/sshd_server_config:/data/pkg/data-backup.tgz}
```