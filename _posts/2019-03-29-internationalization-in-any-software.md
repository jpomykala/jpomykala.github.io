---
layout: post
title: "Internationalization in any software done right"
categories: [software-development, internationalization, i18n, react, software, startup]
---

![image](https://thepracticaldev.s3.amazonaws.com/i/8stvdc832sfg94vu1cuv.jpeg)

Are you struggling with keeping your app translations up to date and your dreams are filled with empty keys to translate? No worries, we know that feeling very well. To make the whole localisation process easier, we created a simple and effective solution which will improve the quality of translations and help you keep them in track! 

# How does it work?

SimpleLocalize is a s cloud oftware created to help developers manage translations. You can use our import tool to upload all keys from your project to our software, translate them in the easy and clean spreadsheet on SimpleLocalize and that’s all you have to do! Translated keys will be synchronised automatically with your project and vice versa, when you add a new key, it will appear in SimpleLocalize cloud ready to add the translation. You can use the CLI to manage keys and translations automatically or you can import and export files manually (we support multiple formats, like React-Intl, GatsbyJS, FormatJS and of course Excel and CSV files). Sounds interesting?

![header](https://thepracticaldev.s3.amazonaws.com/i/1kka5fqpluej6ybyxfws.png)

# i18n keys auto-discovery feature 

SimpleLocalize is created to automatise your work and help you keep all translations up to date. You can create multiple projects and shared them with your team or give access to selected projects and languages to translators team. When their job is completed, you can save the project and synchronise it with your app. You can easily check the progress of the project translations, track the missing keys and use as many languages as you want. Make your app available worldwide thanks to the professional and  always  updated translations!

# How to start?

First, head to SimpleLocalize.io and sign up (there is a 7 day free trial waiting for you!) to get an account. Create a new project by adding its name. Select the project type which will define the way the translations will be exported, for example, just CDN (Content Delivery Network)).

![create-project](https://thepracticaldev.s3.amazonaws.com/i/hxkq4x1uqtoxnx0qwuaw.png)


Open the project and go to Settings. Download configuration properties with the one click on the button Download CLI Properties:

![simplelocalize-download-credentials](https://thepracticaldev.s3.amazonaws.com/i/lsdwjx92wt2aqlyaqpok.png)

Downloaded file will looks like follows:
![credentials](https://thepracticaldev.s3.amazonaws.com/i/mm7odn6xvepggg0g2ymb.png)

Save the file in your project root directory.

Next, run the import tool in your project directory using the command line:

```bash
curl -sL https://cdn.simplelocalize.io/cli/simplelocalize | bash
```
It will search for the keys in your project directory according to `projectType` defined in the configuration file and send them to the SimpleLocalize cloud where you can easily check how may translations are missing and manage them in one place.

Done!

Add a translation and publish the changes to check how the keys are updated in your project! On click on Publish button all translations are sent to your frontend app through S3. With the integration of SimpleLocalize cloud the changes made in your project and then in the SL cloud are updated immediately in the final app.

![simplelocalize-spreadsheet](https://thepracticaldev.s3.amazonaws.com/i/p6vokj8q5zlyrz2mzr7m.png)

Ready to start?

Give it a try and test the new localisation tool we have created. It is designed to help you to keep all translations up to date, easily manage new keys and translations changes and work on multiple project at the same time, in one place! Integrate you web or mobile app with SimpleLocalice and work with your clients efficiently without excel spreadsheets and translations files, instead use the clean and user-friendly interface of SimpleLocalize and control the localisation process in much pleasant way.

If you are interested in the cooperation with us or have any idea or questions about the project or its usage, contact us on contact@simplelocalize.io
