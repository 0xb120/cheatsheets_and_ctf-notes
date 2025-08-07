---
title: Hacklog v1 - Anonimato
author:
  - Stefano Novelli
reading-status: Finished
genre:
  - Computers
publishdate: 2019-01-01
pages: 263
date_read: 2017-03-03
rating: ⭐⭐⭐
cover: https://m.media-amazon.com/images/I/41XgWWrDZ6L.jpg
tags:
  - GPG
  - PGP
---
>[!summary]
>How to stay anonymous: OS, VMs, LiveOS, Fingerprint, VPNs & Tor, Clearnet and Deep Web, HTTPS, Cookies, JavaScript, Flash, Java, ActiveX, WebRTC, Data security, crypto, PGP, GPG, VeraCrypt, TrueCrypt, Stego, Steganography, data backups, metadata, data shredding, data recovery, online identity and online payment

# GPG & PGP

On linux, install `gnupg`:
```sh
apt install gnupg
gpg --version
```

General command:
```sh
# Create a new key pair
gpg --gen-key

# Export your public/private key
gpg --export --armor youremail@example.com > mypubkey.asc
gpg --export-secret-keys --armor [key_id] > private-key.asc

# Import others public keys
gpg --import pubkey.asc

# List public keys in your keyring
gpg --list-keys

# List private keys
gpg --list-secret-keys

# Encrypt a file
gpg --encrypt --recipient Pranab filename.txt  
gpg --encrypt --recipient 'Pranab' filename.txt  
gpg --encrypt --recipient 'pranab@example.com' --recipient 'Sam' filename.txt
# Armor text
gpg --encrypt --armor --recipient 'pranab@example.com' filename.txt
gpg --output filename.asc --encrypt --armor --recipient 'pranab@example.com' filename.txt

# Decrypt
gpg --decrypt filename.txt.gpg  
gpg filename.txt.gpg  
gpg --decrypt filename.txt.gpg > filename-copy.txt

# Symmetric encryption
gpg -c file.txt  
# with more options  
gpg --symmetric \  
--armor \  
--cipher-algo AES256 \  
--output file.enc \  
file.txt
# Symmetric decryption
gpg -d file.txt.gpg > file-copy.txt
```
