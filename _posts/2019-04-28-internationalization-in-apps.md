---
layout: post
title: 🤖 Internationalization tool for yahoo/react-intl
categories: [software-development, internationalization, i18n, react, software, startup]
---

Are you struggling with keeping your app translations up to date, and your dreams are filled with empty keys to translate? No worries, we know that feeling very well. To make the whole localisation process easier, we created a simple and effective solution which will improve the quality of translations and help you keep them in the track!

> This blog post is outdated. Please head to [SimpleLocalize Blog](https://simplelocalize.io/blog/) to check out the most recent blog posts about [React Localization](https://simplelocalize.io/tag/react-localization/). 

![cli](https://thepracticaldev.s3.amazonaws.com/i/8stvdc832sfg94vu1cuv.jpeg)

# How does it work?

**SimpleLocalize is a tool created to help developers manage translations in their apps.** You can use import tool to upload all keys from your project to a cloud editor, translate them in the easy and clean spreadsheet and that’s all you have to do! Translated keys will be synchronised automatically with your project and vice versa, when you add a new key, it will appear in SimpleLocalize cloud ready to add the translation. You can **use SimpleLocalize-CLI to find internationalization keys in the source code automatically** or you can import and export files manually, we support multiple formats, like [yahoo/react-intl](https://github.com/yahoo/react-intl), Android XML String resrouces, iOS, Jekyll, Java properties and of course Excel and CSV files. Sounds interesting?

![flow](https://thepracticaldev.s3.amazonaws.com/i/ui8ddirj973pcujlwdsj.jpg)

# i18n keys auto-discovery feature 

**SimpleLocalize is created to automatise your work and help you keep all translations up to date.** You can create multiple projects and **share them with your team** or give access to selected projects to translators team. When their job is completed, you can **publish changes to the CDN**, refresh your deployed application and voilà! Everything is translated now! You can easily check the progress of the project translations, track the missing keys and use as many languages as you want. Make your app available worldwide thanks to the professional and always updated translations.


# How to start?

First, head to SimpleLocalize.io and [sign up to get an account](https://app.simplelocalize.io/register), the basic plan it's free, and allows you to handle medium sized translation repository. **Create a new project** by adding its name. **Select the project type** which will define the way the translations will be exported, for example, just CDN is the most common for web applications like ReactJS with [yahoo/react-intl](https://github.com/yahoo/react-intl).


![create-project](https://thepracticaldev.s3.amazonaws.com/i/hxkq4x1uqtoxnx0qwuaw.png)


**Open the project and go to *Settings*.** Download configuration properties with the one click on the button *Download CLI Properties*:

![credentials-section.png](/assets/2019-04-28/credentials-section.png)

in downloaded file set `projectType:` property to `yahoo/react-intl`  and it should look now like follows:
```yaml
apiKey: <PROJECT_UPLOAD_TOKEN>
projectType: yahoo/react-intl
```

[*Read more about CLI `projectType` property*.](https://simplelocalize.gitbook.io/simplelocalize/supported-libraries) Save the file in your project root directory. Next, run the CLI tool in your project directory using the command line:

```bash
curl -s https://get.simplelocalize.io | bash
```
It will search for the keys in your project directory according to `projectType` defined in the configuration file and send them to the SimpleLocalize cloud where you can easily check how may translations are missing and manage them in one place.


![spreadsheet](https://thepracticaldev.s3.amazonaws.com/i/p6vokj8q5zlyrz2mzr7m.png)


Done, you successfully configured the project!

Add translations and publish changes to check how the keys are updated in your project! **When you click *Publish* button all translations will be sent to your frontend app through the CDN**, it works even when the application is deployed. That means you can change text on page without rebuilding and redeploying the whole application. Pretty useful, huh?

Now you can fetch translations using url like follows:

[https://cdn.simplelocalize.io/:projectToken/_latest/:languageKey](https://app.gitbook.com/@simplelocalize/s/simplelocalize/api/generic-cdn)

or use [`react-intl-simplelocalize` library](https://simplelocalize.gitbook.io/simplelocalize/integrations/reactjs-integration) for ReactJS applications.

![project-list](https://thepracticaldev.s3.amazonaws.com/i/fr5kjtvxd1ftr45188ys.png)

# Ready to start?

Give it a try and test the new internationalisation tool we have created. It is designed to help you to keep all translations up to date, easily manage new keys and translations changes and work on multiple project at the same time, in one place. Integrate you web or mobile app with SimpleLocalize and work with your clients efficiently without Excel spreadsheets and translations files, instead use the clean and user-friendly interface of SimpleLocalize and control the i18n process in much pleasant way. 
[Create account, no credit card required!](https://app.simplelocalize.io/register)


