---
layout: post
title: "SQL for dummies"
categories: thcon
order: 5
flag: "THCon21{eA3y*QL_1nject0R}"
challenge_type: "intro"
event: "thcon21"
---
## SQL for dummies

### Overview

**Category:** Intro /
**Points:** 10 (dynamic) / 
**Solves:** 292

**Description:**

>A lazy admin thinks is login page is secure, show him the contrary !
>
>    - http://remote1.thcon.party:10705
>    - http://remote2.thcon.party:10705

**Creator(s) :** jrjgjk (Discord : guilhem#8743) and 23euros

### Writeup

Simplest of SQLi (SQL injections), nothing particular to say.

Type `'or 1=1;--` as the username, and anything as the password.

{%- include flag.html flag=page.flag -%}