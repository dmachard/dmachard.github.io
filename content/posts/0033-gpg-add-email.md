---
title: "GPG: add an email to an existing key"
summary: "This post details how to add an email to an existing GPG key"
date: 2021-11-10T00:00:00+01:00
draft: false
tags: ['gpg', 'howto']
---

# GPG: add an email to an existing key

This post details how to add an email to an existing GPG key

## How to

List your existing GPG keys.

```bash
gpg --list-secret-keys --keyid-format=long
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/home/denis/.gnupg/pubring.kbx
------------------------------
sec   rsa4096/123FFFFFFF456 2021-06-09 [SC]
      1294...........BE886
uid                 [ultimate] John Doe <jdoe@gmail.com>
ssb   rsa4096/C07....BEA 2021-06-09 [E]
```

In this example, the GPG key ID is **123FFFFFFF456**

Enter in your gpg prompt for your key and type adduid 

```bash
gpg --edit-key 123FFFFFFF456
gpg> adduid 
Real name: John Doe
Email address: 123456+jdoe@users.noreply.github.com
Comment: GitHub key
You selected this USER-ID:
    "John Doe (GitHub key) <123456+jdoe@users.noreply.github.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O

sec  rsa4096/123FFFFFFF456
     created: 2021-06-09  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/C0.....BEA
     created: 2021-06-09  expires: never       usage: E   
[ultimate] (1)  John Doe <jdoe@gmail.com>
[ unknown] (2). John Doe (GitHub key) <123456+jdoe@users.noreply.github.com>
```

Finally save the changes

```bash
gpg> save
```

Now you can export you new GPG key

```bash
gpg --armor --export <MY_KEY_ID>
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg
....
mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg
-----END PGP PUBLIC KEY BLOCK-----
```
