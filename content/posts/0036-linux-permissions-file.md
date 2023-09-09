---
title: "Manage Linux permissions for users, groups, and others"
summary: "How to"
date: 2021-11-13T00:00:00+01:00
draft: false
tags: ['linux', 'security', 'howto']
---

# Manage Linux permissions for users, groups, and others

How to manage Linux permissions for users, groups and others

## Access level

Display file permission

```bash
$ ls -alrt
total 1
-rwxrwxr--. 1 ansible automation 4 Nov 13 10:57 helloworld.txt
```

- r = read = 4
- w = write = 2
- x = execute = 1

## Identity

```bash
$ ll
total 1
-rwxrwxr--. 1 ansible automation 4 Nov 13 10:57 helloworld.txt
```

[ user = u ] [ group = g ] [ others = o ]

- The user *ansible* has 4+2+1=7 (full access)
- The group *automation* has 4+2+1=7 (full access)
- All others have 4  (read-only)

## Chmod and Chown

Use the command **chmod** to change permission file or directory.

```bash
$ chmod 644 myfile
```

Use the command **chown** to change user and group appartenance.

```bash
$ chown -R user:group /mydirectory/
```
