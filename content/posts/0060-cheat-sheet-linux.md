---
title: "Cheat sheets for Linux, Git, GPG, VIM, and more"
summary: "This page lists some commands for Linux, Git, Ssh, GPG, etc..."
date: 2024-01-07T00:00:00+01:00
draft: false
tags: ['cheat-sheet']
pin: false
---

# Cheat sheets

## Linux

| Cheat sheet | Commands  |
| ------ | --------- |
| list timezone | <pre>timedatectl list-timezones</pre> |
| set new timezone | <pre>sudo timedatectl set-timezone Europe/Paris</pre> |
| update hostname                     | <pre>sudo hostnamectl set-hostname [new_name]</pre> |
| show version ubuntu                 | <pre>lsb_release -a</pre> |
| add static ip with Netplan         | <pre>sudo vim /etc/netplan/01-cfg-ens19.yaml<br/>network:<br/>    ethernets:<br/>    ens19:<br/>        addresses:<br/>        - 172.16.0.1/12<br/>    version: 2<br/>sudo chmod 600 /etc/netplan/*<br/>sudo netplan apply</pre> |
| lists network interfaces with NetworkManager | <pre>nmcli connection show<br>NAME          UUID                                  TYPE      DEVICE<br> WiFi5      79361210-59bd-4a91-a4af-c78634446295  wifi      wlp2s0<br></pre> |
| rename Interface with NetworkManager | <pre>nmcli connection modify "Wired connection 1" connection.interface-name "ens19"</pre> |
| add static IP with NetworkManager | <pre>nmcli con mod <NET_UUID> ipv4.address 192.168.1.2/24<br>nmcli con mod <NET_UUID> ipv4.gateway 192.168.1.1<br >nmcli con mod <NET_UUID> ipv4.method manual<br>nmcli con mod <NET_UUID> ipv4.dns 8.8.8.8<br>nmcli con mod <NET_UUID> autoconnect yes<br>nmcli con down <NET_UUID><br>nmcli con up <NET_UUID></pre> |
| display file permission and ownership | <pre>ls -alrt<br>-rwxrwxr--. 1 ansible automation 4 Nov 13 10:57 helloworld.txt<br><br>r = read = 4<br>w = write = 2<br>x = execute = 1<br>[ user = u ] [ group = g ] [ others = o ]<br>The user *ansible* has 4+2+1=7 (full access)<br>The group *automation* has 4+2+1=7 (full access)<br>All others have 4  (read-only)</pre> |
| change permission file or directory | <pre>chmod 644 myfile</pre> |
| change user and group appartenance | <pre>chown -R user:group /mydirectory/</pre> |
| Extend physical drive partition | <pre># check free space<br>sudo fdisk -l<br># Extend physical drive partition<br>sudo growpart /dev/sda 3 <br># See  phisical drive<br>sudo pvdisplay<br># Instruct LVM that disk size has changed<br>sudo pvresize /dev/sda3</pre> |
| resize logical volume | <pre># View starting LV<br>sudo lvdisplay<br># Resize LV<br>sudo lvextend  -l +100%FREE /dev/ubuntu-vg/ubuntu-lv<br>df -h<br>#Resize Filesystem<br>sudo resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv</pre> |
|  Create partition for New Disk  | <pre>fdisk /dev/sdc</pre> |
| format the disk with mkfs command | <pre>mkfs.ext4 /dev/xvdc1</pre> |
| share file with windows | <pre>sudo apt-get install samba<br>sudo smbpasswd -a denis<br>sudo vim /etc/samba/smb.conf<br>[data]<br>   path = [folder_to_share]<br>   valid users = [user]<br>   read only = no<br>   # guest ok = yes # no auth<br>sudo systemctl restart smbd</pre> |

## Ubuntu desktop

| Cheat sheet | Commands  |
| ------ | --------- |
| install basic tools| <pre>sudo apt install vim net-tools htop vlc</pre> |
| enable ssh server| <pre>sudo apt install openssh-server -y<br >Edit /etc/ssh/sshd_config<br>PasswordAuthentication yes<br>sudo systemctl restart ssh</pre> |
| create USB bootable| <pre>https://etcher.balena.io/#download-etcher</pre> |
| install XRDP | <pre>sudo apt-get install xrdp<br>sudo systemctl enable xrdp<br>setxkbmap fr</pre> |
| quick fix for XRDP and Ubuntu 23.10 |<pre>DesktopVer="$XDG_CURRENT_DESKTOP"<br>SessionVer="$GNOME_SHELL_SESSION_MODE"<br>ConfDir="$XDG_DATA_DIRS"<br>sudo sed -i "4 a #Improved Look n Feel Method\ncat <<EOF > ~/.xsessionrc\nexport GNOME_SHELL_SESSION_MODE=$SessionVer\nexport XDG_CURRENT_DESKTOP=$DesktopVer\nexport XDG_DATA_DIRS=$ConfDir\nEOF\n" /etc/xrdp/startwm.sh</pre> |

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

## Vim

| Cheat sheet | Commands  |
| ------ | --------- |
| Delete specific lines | <pre>:g/<REGEX_PATTERN>/d</pre> |

## Docker

| Cheat sheet | Commands  |
| ------ | --------- |
| Install docker | https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository |

## PowerDNS / pdns-auth

| Cheat sheet | Commands  |
| ------ | --------- |
| Install sqlite db | <pre>sudo apt install sqlite3<br>wget [github_url_pdns]/master/modules/gsqlite3backend/schema.sqlite3.sql<br>sqlite3 pdns.db<br>.read schema.sqlite3.sql<br>.quit<br></pre> |
| list all zones |  <pre>sudo docker compose exec pdns pdnsutil list-all-zones</pre> |
| Create zone |  <pre>sudo docker compose exec pdns pdnsutil create-zone home.</pre> |
| add records |  <pre>pdnsutil add-record home. ns1 A 3600 172.16.0.253<br>New rrset:<br>ns1.home. 3600 IN A 172.16.0.253</pre> |
| Update record |  <pre>pdnsutil replace-rrset home. test A 3600 192.168.1.253<br>Current records for test.home IN A will be replaced<br>New rrset:<br>test.home. 3600 IN A 192.168.1.253</pre> |
