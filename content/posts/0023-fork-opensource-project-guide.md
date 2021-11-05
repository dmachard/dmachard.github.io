---
title: "Contribution guide to open source project - fork and pull request"
date: 2021-10-05T00:00:00+01:00
draft: false
tags: ['fork', 'github']
---

# Table of contents

* [GitHub Fork & Pull Request ](#gitHub-fork-pull-request )

# GitHub Fork & Pull Request 

Fork the project you want to contribute

Clone your fork to your local machine

```bash
git clone git@github.com:USERNAME/FORKED-PROJECT.git
```

Add 'upstream' repo to list of remotes

```bash
git remote add upstream https://github.com/UPSTREAM-USER/ORIGINAL-PROJECT.git
```

Verify the new remote named 'upstream'

```bash
git remote -v
```

Fetch from upstream remote

```bash
git fetch upstream
```

View all branches, including those from upstream

```bash
git branch -va
```

Checkout your master branch and merge upstream

```bash
git checkout master
git merge upstream/master
```

Create a new branch named newfeature (give your branch its own simple informative name)

```bash
git branch newfeature
```

Switch to your new branch

```bash
git checkout newfeature
```

Finally make changes in the code