---
layout: post
title: "Your browser is managed"
tags: [Chrome]
date: 2020-06-12
---

Why my Chrome browser show "Your browser is managed" even I don't have organization doing so?  How to remove it?

* regedit

HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Google\Chrome

Just remove all elements under this entry. And restart Chrome.

