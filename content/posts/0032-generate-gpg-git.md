---
title: "GPG: how to generate a key to sign commits with Git"
summary: "This post details how to generate a GPG key to sign commits with Git"
date: 2021-11-10T00:00:00+01:00
draft: false
tags: ['git', 'gpg']
---

# GPG: how to generate a key to sign commits with Git

This post details how to generate a GPG key to sign commits with Git

## Generate a new GPG key

Generate a GPG key pair an enter user ID information.

```bash
gpg --full-generate-key
```

List GPG keys. 

```bash
gpg --list-secret-keys --keyid-format=long
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
/home/denis/.gnupg/pubring.kbx
------------------------------
sec   rsa4096/<MY_KEY_ID> 2021-06-09 [SC]
      1294...........BE886
uid                 [ultimate] John Doe <jdoe@gmail.com>
ssb   rsa4096/C07....BEA 2021-06-09 [E]
```

## Add a GPG key to Github

Copy your GPG key, beginning with -----BEGIN PGP PUBLIC KEY BLOCK----- and ending with -----END PGP PUBLIC KEY BLOCK----

```bash
gpg --armor --export <MY_KEY_ID>
-----BEGIN PGP PUBLIC KEY BLOCK-----

mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg
....
mQINBGDBBhIBEADD/m4EK+XFiW20rE8fhLgom+zI/eExjaTUbLrLPj2q6SxxX2rg
-----END PGP PUBLIC KEY BLOCK-----
```

and Paste-it in your Github [account](https://github.com/settings/keys)

## Configure Git to use your GPG key

```bash
git config --global user.signingkey MY_KEY_ID
```
