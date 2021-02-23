---
layout: post 
title: "💼️ Why it's a good idea to keep e-mail registration in 2021"
tags: [java, startup, devlog]
---

When I started working on [my app for translation management](https://simplelocalize.io) in 2018, I knew it would be good to support single sign in option (SSO) with GitHub.

The service was created with intention that it will be used by developers, and developers have GitHub accounts! I even
advertised it as "Localization for Software Developers" or "developer-friendly localization". Although, it didn't get too much [attention on Hacker News](https://news.ycombinator.com/item?id=19823169), 
but hey! Now I can put Hacker News logo on my landing page and say "Featured on". 👌

![simplelocalize featured on hackernews](/assets/2021-02-12/hacker-news.png)

As the time goes I implemented good old e-mail registration because I had ready to use code and [open-source library for
Java Spring Boot](https://github.com/jpomykala/spring-higher-order-components) to send e-mails. The most of the work was done.

Thanks to OAuth library for Spring Boot, I also added Google account login with ease. I only needed to add a few more lines in configuration file and voilà! 
Visitors are able to create accounts by Google Profile. I couldn't add LinkedIn due to some problems, so I just left what I got in config and gave up. 😅

![simplelocalize sso option](/assets/2021-02-12/spring-oauth.png)

Eventually, I launched SimpleLocalize in late 2018. I started posting this project everywhere, ProductHunt, Reddit, dev.to, Medium and more.

![simplelocalize sketch draft](/assets/2021-02-12/first-simplelocalize-draft.png)

After 2 years of application existence I gathered over 800 users and surprisingly most of them used Google to register.
It's about 550 accounts which used Google to create their accounts. It was a surprise for me. Only 20 people chosen GitHub (including me 😄),
and the rest used e-mail, or their accounts has been created by project admins.
> SimpleLocalize offers project sharing where you can add team member by e-mail and then account is automatically created for them.
> Their accounts are always created using 'e-mail' method. 

![top selected register options](/assets/2021-02-12/pie-chart.png)


I noticed that every person who used GitHub or Google to register, after trying the service wanted 
to move its account to business e-mail. The second option was that they were creating a new account with company e-mail, 
and asking me to move their projects to the new account. This is something what should interest every bootstrapper. **Don't kick out 
the e-mail registration, people still use it to create accounts using business e-mails instead personal accounts!**

In conclusion, my project is now targeted to more professional organizations and small/medium agile team. 
I needed to change my plans, to make sure I can provide support fast enough. 💪

![simplelocalize plans](/assets/2021-02-12/plans.png)

I also plan to add Microsoft login in the near feature.
Many corporate employees use Outlook and Microsoft ecosystem. 
They will more encouraged to create an account in the system and try it out. 
They will not have to remember a password to e-mail accounts and use their personal accounts. 
In the further feature it would be great to add more corporate solutions like OpenID. 
