---
layout: post
title: "🌱 Sending e-mails based on user activity"
tags: ['marketing', 'startup', 'customer base', 'devlog']
---



## High quality backlinks

- Wikipedia
- Ask other bloggers about a review, or a post
- Ask other bloggers to write a post on your blog, so they can link to their blog


## Important meta headers (copy-paste-replace)

I use React-Helmet to put meta tags in my GatsbyJS landing pages

```html
<title>{title}</title>
<meta name="description" content={description}/>
<meta name="thumbnail" content={thumbnail}/>

<meta name="robots" content="index, follow"/>
<meta name="googlebot" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1"/>
<meta name="bingbot" content="index, follow, max-snippet:-1, max-image-preview:large, max-video-preview:-1"/>
<meta name="keywords" content={keywords}/>

<meta name="viewport" content="width=device-width, initial-scale=1"/>

<!-- Tell Google that I'm focusing on this link -->
<link rel="canonical" href="https://jpomykala.com/">

<!-- Apple TouchBar icon -->
<link color="#1E232C" href={touchbarIcon} rel="mask-icon"/>

<!-- Facbook -->
<meta property="og:title" content={title}/>
<meta property="og:image" content={facebookImage}/>
<meta property="og:site_name" content="SimpleLocalize"/>
<meta property="og:description" content={description}/>
<meta property="og:locale" content="en_US"/>
<meta property="og:type" content="website"/>

<!-- Twitter -->
<meta name="twitter:description" content={description}/>
<meta name="twitter:card" content="summary"/>
<meta name="twitter:title" content={title}/>
<meta name="twitter:image" content={twitterImage}/>
```

## Proper use of h1, h2, h3

Here is the hierarchy of header tags:

<h1></h1> – usually reserved for webpage titles.
<h2></h2> – highlights the topic of the title.
<h3></h3> – reflects points in regards to the topic.
<h4></h4> – supports points from <h3>.
<h5></h5> – not often used, but great for supporting points of <h4>.

## Add <img> alt attribute 

Instead of

```html
<img src="plant.jpg"/>
```

use:

```html
<img src="plant.jpg" alt="plant with yellow leaves"/>
```

Search engine will now understand what is on the image. Do not put words
like "logo", "image", "picture". It's obvious it's an image, you put i in the <img> tag 😄

## Simple slug

Instead of 

`/seo/category/blog/post?id=12304`

use:

`/blog/what-is-hreflang`

## Every page should be as good as your landing

Example, here is my SimpleLocalize landing page: https://simplelocalize.io and here is my subpage for react localization: https://simplelocalize.io/react-localization
Once somebody will be looking for 'react localization' phrase in Google, then he will find out the second link and click it

## Be everywhere
It's hard to do, but it's important. Twitter, Facebook, alternative tool websites, blogs, medium, dev.to, hackernews. It's hard to know which of it will work the best for 
your webpage. You can measure it using Google Analytics or [SimpleAnalytics](https://simpleanalytics.com).

## Tools

- Ahrefs
- Google Search Console

## Do not translate slugs

It's simple as that, Google won't rank your web page higher because of this. It's recommended to use language key before
a slug. Like this `/en/contact` and accordingly `/pl/contact`. This is okay.
