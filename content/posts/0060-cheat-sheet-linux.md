---
title: "Cheat sheets for Linux, Git, GPG, VIM, and more"
summary: "This page lists some commands for Linux, Git, Ssh, GPG, etc..."
date: 2024-01-07T00:00:00+01:00
draft: false
tags: ['cheat-sheet']
pin: false
---

# Cheat sheets for Linux, Git, Sshd, GPG and more

## Linux

| Cheat sheet | Commands  |
| ------ | --------- |
| update hostname                     | <pre>sudo hostnamectl set-hostname [new_name]</pre> |
| show version ubuntu                 | <pre>lsb_release -a</pre> |
| add static ip with Netplan         | <pre>sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>network:<br/>    ethernets:<br/>    ens19:<br/>        addresses:<br/>        - 172.16.0.1/12<br/>    version: 2<br/>sudo chmod 600 /etc/netplan/*<br/>sudo netplan apply</pre> |
| lists network interfaces with NetworkManager | <pre>nmcli connection show<br>NAME          UUID                                  TYPE      DEVICE<br> WiFi5      79361210-59bd-4a91-a4af-c78634446295  wifi      wlp2s0<br></pre> |
| rename Interface with NetworkManager | <pre>nmcli connection modify "Wired connection 1" connection.interface-name "ens19"</pre> |
| add static IP with NetworkManager | <pre>nmcli con mod <NET_UUID> ipv4.address 192.168.1.2/24<br>nmcli con mod <NET_UUID> ipv4.gateway 192.168.1.1<br >nmcli con mod <NET_UUID> ipv4.method manual<br>nmcli con mod <NET_UUID> ipv4.dns 8.8.8.8<br>nmcli con mod <NET_UUID> autoconnect yes<br>nmcli con down <NET_UUID><br>nmcli con up <NET_UUID></pre> |
| lists network interfaces with NetworkManager | <pre></pre> |
| display file permission and ownership | <pre>ls -alrt<br>-rwxrwxr--. 1 ansible automation 4 Nov 13 10:57 helloworld.txt<br><br>r = read = 4<br>w = write = 2<br>x = execute = 1<br>[ user = u ] [ group = g ] [ others = o ]<br>The user *ansible* has 4+2+1=7 (full access)<br>The group *automation* has 4+2+1=7 (full access)<br>All others have 4  (read-only)</pre> |
| change permission file or directory | <pre>chmod 644 myfile</pre> |
| change user and group appartenance | <pre>chown -R user:group /mydirectory/</pre> |

## SSH

| Cheat sheet | Commands  |
| ------ | --------- |
| generating a new SSH public and private key | <pre>ssh-keygen -b 4096</pre> |
| copy the public key to remote server | <pre>ssh-copy-id username@remote_host</pre> |
| disabling Root Login in SSHD | <pre>sudo nano /etc/ssh/sshd_config<br>PermitRootLogin no<br>sudo service sshd restart</pre> |
| disabling Password Authentication on SSHD | <pre>sudo nano /etc/ssh/sshd_config<br>PasswordAuthentication no<br>sudo service sshd restart</pre> |

## Git

| Cheat sheet | Commands  |
| ------ | --------- |
| install git | <pre>sudo apt install git</pre> |
| config client | <pre>git config --global user.name <USER_NAME><br>git config --global user.email <USER_EMAIL></pre> |
| use your GPG key | <pre>git config --global commit.gpgsign true<br>git config --global user.signingkey <KEY_ID></pre> |

## GPG

| Cheat sheet | Commands  |
| ------ | --------- |
| list GPG keys | <pre>gpg --list-secret-keys --keyid-format=long<br>sec   rsa4096/<KEY_ID> 2021-06-09 [SC]</pre> |
| generate a key | <pre>gpg --full-generate-key</pre> |
| update an existing key | <pre>gpg --edit-key <KEY_ID></pre> |
| import key | <pre>gpg --import "gpg_private.key"</pre> |
| export GPG key | <pre>gpg --armor --export <KEY_ID>-----BEGIN PGP PUBLIC KEY BLOCK-----<br>mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg<br>...<br>mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg<br>-----END PGP PUBLIC KEY BLOCK-----</pre> |

## VIM

| Cheat sheet | Commands  |
| ------ | --------- |
| Delete specific lines | <pre>:g/<REGEX_PATTERN>/d</pre> |
