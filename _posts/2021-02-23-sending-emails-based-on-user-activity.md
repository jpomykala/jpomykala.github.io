---
layout: post 
title: "🌱 Sending e-mails based on user activity"
tags: ['marketing', 'startup', 'customer base', 'devlog']
---

Okay, it's time to introduce user activity mailing! I was always sceptical about sending SPAM to trial users. Recently,
I tried a few services which I really wanted to use. I simply didn't have time to do that. I hit a register button and forget. After a few days I started getting a massive number of emails.

It worked 😄

I was getting e-mails as reminders to do this and that. When a weekend came, I had a few hours of free time to deep
dive into those services. Meet the internals, configure it to add it to the CI/CD pipeline.

In SimpleLocalize I was providing really minimalistic e-mails. Simple "Hi", some text, link, "Thank you" and "Your SimpleLocalize Team".

![example simplelocalize e-mail](https://jpomykala.com/assets/2021-02-23/email.png)

I was determined enough to put a user-friendly name like "SimpleLocalize.io" instead just "contact@simplelocalize.io" 😉
The first service which came up to my mind was Mailchimp. Popular, well-known service marketing tool. For me it's overcomplicated and stuffed 
with too many functions, but I gave it a chance.

![mailchimp request](https://jpomykala.com/assets/2021-02-23/mailchimp-request.png)

After some digging I found out that you can send promotional e-mails to 2000 contacts for free, but not transactional e-mails. What is a transactional e-mail? It's noting else than sending one e-mail to one person. This feature
is provided by Mandrill, and it's paid.

![mandrill paid extension](https://jpomykala.com/assets/2021-02-23/mandrill.png)

Okay, so let's organize it on my own. I already had configured Amazon SNS for sending e-mails needed for [e-mail registration](https://jpomykala.com/2021/02/12/why-email-login-matters-and-sso-is-important).

Creating an e-mail template is the worst pain in the modern software development. They are created mostly as html tables, with a lot of <tr> and <td> tags.

![mjml template creation](https://jpomykala.com/assets/2021-02-23/mjml-code.png)

I know a nice tool for e-mail templates. [MJML](https://mjml.io) is a super simple and easy to use framework for e-mail template creation. It's based on React ☺️ and generated e-mail templates
work on almost all kind of devices and e-mail programs. Created e-mail template looks good also on mobiles. 📱

I made a very simple template with [mjml](https://mjml.io).

![email sending worker in Java and Spring Framework](https://jpomykala.com/assets/2021-02-23/email-sending-worker.png)

I added a simple worker code which runs every 2 hours to check to whom I should send e-mails. 
I save information if user already received the e-mail. In MongoDB, I keep user e-mail, date, and e-mail type.
Furthermore, I also created an API endpoint to unsubscribe from those e-mails.

In the future, I will also translate those e-mails. Unfortunately, I don't keep information about user locale in the database,
so I cannot do it right now.

![sample simplelocalize e-mail created with mjml template](https://jpomykala.com/assets/2021-02-23/email-sending-worker.png)

Done! 🌱

![example email created with mjml](https://jpomykala.com/assets/2021-02-23/sample-simplelocalize-email.png)


What else I need to do?
- add 'list-unsubscribe' header to e-mails
- save information about user locale
- translate e-mails
