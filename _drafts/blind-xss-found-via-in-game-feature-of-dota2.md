---
comments: true
title: "Blind XSS Found Via In-Game Feature of Dota 2 \U0001F3AE"
date: 2022-03-31 00:00:00 +0700
category:
- bugbounty
tags:
- xss
- web
- dota2
private: false

---
The finding started when Dota 2 is announcing the new feature for the battle pass owner that is being able to create a guild in the game.

> **Guild Updates**  
>   
> We reintroduced Guilds during the Battle Pass, and we were happy to see how many players participated in Guilds and shared their feedback with us. We are keeping the existing Guilds, but many of the rewards received only worked in the context of the Battle Pass. We've redesigned those features so that you can enjoy playing with your guild all the time.
>
> \- [https://www.dota2.com/newsentry/3066366095799664170](https://www.dota2.com/newsentry/3066366095799664170 "https://www.dota2.com/newsentry/3066366095799664170")