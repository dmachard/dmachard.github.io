---
title: "Auto start Virtualbox VM on boot"
summary: "This post details how to start automatically a VM on boot."
date: 2019-02-08T00:00:00+01:00
draft: false
tags: ['vm', 'windows']
---

# Auto start Virtualbox VM on boot

This post details how to start automatically a VM on boot.

## Guide

Create a shortcut.
   - Open virtualbox
   - highlight the VM you want to startup
   - click "Machine" -> "Create Shortcut on Desktop"
   
Add to windows startup
   - Start -> Run
   - type in "shell:startup"
   - copy & paste shortcut in here.