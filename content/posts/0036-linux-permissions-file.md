---
title: "How to manage Linux permissions for users, groups, and others"
date: 2021-11-13T00:00:00+01:00
draft: false
tags: ['linux', 'security']
---

How to manage Linux permissions for users, groups and others

# Table of contents

* [Access level](#access-level)
* [Identity](#identity)
* [Chmod](#chmod)

# Access level

```bash
$ ll
total 1
-rwxrwxr--. 1 automation automation 4 Nov 13 10:57 helloworld.txt
```

- r = read = 4
- w = write = 2
- x = execute = 1

# Identity

```bash
$ ll
total 1
-rwxrwxr--. 1 automation automation 4 Nov 13 10:57 helloworld.txt
```

- -rwxrwxr-- = [ user = u ] [ group = g ] [ others = o ]

The user has 4+2+1=7 (full access)
The group has 4+2+1=7 (full access)
All others have 4  (read-only)

# Chmod

Use the command **chmod** to change permission file or directory.

```bash
$ chmod 644 file2
```