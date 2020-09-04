---
layout: post
title: Why handling i18n in software projects is tough
categories: [i18n, localization, intl, internationalization]
---

![i18n intl post header](https://dev-to-uploads.s3.amazonaws.com/i/xp9qorzsqw4cblsg37l4.jpg)

Have you ever thinking about adding more languages to your application? Does it make sens to do it? Will you get more customers? Or will it make your existing customers happier? Or maybe you would like to look more professional than your competitors? Let's make it clear - yes, your app will look much more profesional if you will give your clients more languages to choose besides English which is language of the Internet by default. Every big tech company provides more than English language in their apps. 

# Background history

When I was creating my first big B2C web application I knew I will need to support more languages than only English. I wanted to reach more mature people who often don't know English at all. Application initially was designed to support 5 languages: 
- English, because it's by default the Internet language
- Polish, because I'm from Poland and I wanted to reach people like my parents who would like to use this app in their first language
- Spanish, because my girlfriend knows this language very well
- Italian, because we planned to add places from Italy
- French, because my girlfriend would like to learn this language 👩‍🎨

I started working on the coding side and my girlfriend was taking care about everything else including translations. We noticed very quickly that i18n... is though... 

![i18n is though](https://dev-to-uploads.s3.amazonaws.com/i/68b2thkwc251ymkeqo3c.jpg)

I was using ReactJS to create [PlaceFlare.com](https://PlaceFlare.com) so I decided to go with [yahoo/react-intl](https://www.npmjs.com/package/react-intl) from [FormatJS.io](https://FormatJS.io) as my primary i18n library. It was very easy to configure and to use. The problem was with handling i18n keys and translations for more than one language.

# Why maintaining i18n is so hard

Basically it was very hard to handle and maintain all i18n keys because every language translations were packed into separate JSON objects or even separate JSON files like en.json, it.json and so on.

![i18n translations file en.json](https://dev-to-uploads.s3.amazonaws.com/i/sgmeka8xiapgm4ccwooh.png)

Making changes was hell! When I needed to update an i18n key in code, I had to do the changes in every translation file, and in the end I needed to update the translation itself to match it.

# How I solved problem which I created

I came up with the simple software solution. We started keeping translations in Excel files like shown below.

![i18n app translations in Excel file](https://dev-to-uploads.s3.amazonaws.com/i/t1f8jbdu3t5bbjdd0uzq.png)

So I provided a small Java app to convert such Excel file to format required by FormatJS library. More problems come up later, like finding i18n in application code, also i18n key changes were still a nightmare, keeping one Excel file which should be always up-to-date, more and more problems to solve.

# How I improved my clunky solution

Very quickly I turned everything what I learned into small simple SaaS project which I called [SimpleLocalize.io](https://simplelocalize.io) I think I solved most of the problems with i18n app translations.

![simplelocalize project list](https://simplelocalize.io/static/ui-2fee8e735b014d8baea37d93c6108a41.png)

The biggest advantage of [SimpleLocalize.io](https://simplelocalize.io) is that you can use any i18n library you want. [SimpleLocalize CLI](https://github.com/simplelocalize/simplelocalize-cli) will extract i18n keys from application source code, push them to the cloud where you can edit it and publish to the CDN which will make it easy to fetch and use in app. 

![hide the pain harold](https://dev-to-uploads.s3.amazonaws.com/i/xiv7rhog74tcsz6pizwx.jpg)
