---
title: "My linux cheat sheet"
summary: "This cheat sheet lists some Linux commands"
date: 2024-01-07T00:00:00+01:00
draft: false
tags: ['cheat-sheet']
pin: false
---

# My linux cheat sheet

## Ubuntu

| Cheat sheet | Commands  |
| ------ | --------- |
| update hostname                     | <pre>sudo hostnamectl set-hostname [new_name]</pre> |
| show version ubuntu                 | <pre>lsb_release -a</pre> |
| add static ip with netplan          | <pre>sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>network:<br/>    ethernets:<br/>    ens19:<br/>        addresses:<br/>        - 172.16.0.1/12<br/>    version: 2<br/>sudo chmod 600 /etc/netplan/*<br/>sudo netplan apply</pre> |
| display file permission and ownership | <pre>ls -alrt<br>-rwxrwxr--. 1 ansible automation 4 Nov 13 10:57 helloworld.txt<br><br>-r = read = 4<br>-w = write = 2<br>-x = execute = 1<br>-[ user = u ] [ group = g ] [ others = o ]<br>-The user *ansible* has 4+2+1=7 (full access)<br>-The group *automation* has 4+2+1=7 (full access)<br>-All others have 4  (read-only)</pre> |
| change permission file or directory | <pre>chmod 644 myfile</pre> |
| change user and group appartenance | <pre>chown -R user:group /mydirectory/</pre> |

## SSH

| Cheat sheet | Commands  |
| ------ | --------- |
| Generating a new SSH public and private key | <pre>ssh-keygen -b 4096</pre> |
| Copy the public key to remote server | <pre>ssh-copy-id username@remote_host</pre> |
| Disabling Root Login in SSHD | <pre>sudo nano /etc/ssh/sshd_config<br>PermitRootLogin no<br>sudo service sshd restart</pre> |
| Disabling Password Authentication on SSHD | <pre>sudo nano /etc/ssh/sshd_config<br>PasswordAuthentication no<br>sudo service sshd restart</pre> |

## Git

| Cheat sheet | Commands  |
| ------ | --------- |
| use your GPG key | <pre>git config --global user.signingkey <KEY_ID></pre> |

## GPG

| Cheat sheet | Commands  |
| ------ | --------- |
| List GPG keys | <pre>gpg --list-secret-keys --keyid-format=long<br>sec   rsa4096/<KEY_ID> 2021-06-09 [SC]</pre> |
| Update an existing key | <pre>gpg --edit-key <KEY_ID></pre> |
| export GPG key | <pre>gpg --armor --export <KEY_ID>-----BEGIN PGP PUBLIC KEY BLOCK-----<br>mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg<br>....<br>mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg<br>-----END PGP PUBLIC KEY BLOCK-----</pre> |
