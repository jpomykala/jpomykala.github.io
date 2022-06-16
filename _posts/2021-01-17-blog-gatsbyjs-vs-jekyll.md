---
layout: post 
title: "☀️ Blog: Jekyll vs GatsbyJS"
tags: [jekyll, design, gatsbyjs, javascript]
---

Do you know that feeling when you are adjusting blog CSS instead of writing posts? I spent countless hours on optimizing landing pages for [my projects](https://jpomykala.com/#projects), and [my personal blog](https://jpomykala.com) using GatsbyJS. 
Eventually, I've never published my blog on [GatsbyJS](https://gatsbyjs.com). I used [Jekyll](https://jekyllrb.com), and here are my thoughts.

I thought that a such powerful tool like GatsbyJS and React syntax would be the best match for me. I'm familiar with React, 
and I hoped that work will be a pleasure. **Great SEO, plugins support, npm libraries, favourite syntax, cutting-edge technologies, endless possibilities!**
Looking at this from a-few-years-experience distance, I must admit it wasn't worth it. 
GatsbyJS is great, but his only advantage over all other competitors is that you can use React components, and that's all. 
I'm using Gatsby for over 3 years now, and I must say **it's an over-engineered, complex, hard to fix static page generator.**
- If it works as intended, then it's great software.
- If you need to do something, then it's a nightmare. 
- Error logs are not clear what was wrong, sometimes you need to remove `node_modules` couple of times to fix it. 
- Gatsby plugin order matters! I didn't go deep into that, but it's hard to figure out which plugin should go first to not break the next ones. 
- The biggest issue is GraphQL which makes your work annoying!

![gatsby develop](/assets/2021-01-17/gatsby-develop.jpeg)

Often, I was having a problem with page generation from GraphQL queries. When I wanted to add tags to every post in [SimpleLocalize blog which I lead](https://simplelocalize.io/blog/),
I spent half of a day on it! I was using [official docs](https://www.gatsbyjs.com/docs/adding-tags-and-categories-to-blog-posts/) which were outdated... ⛈ 
GraphQL support is a powerful tool but very annoying to use.

If you want to play with React, then use it. If you want to do your job quickly without frustration, then use Jekyll. I didn't
try other static page generators because they are less popular. I was afraid of that I will be stuck at some point with a
problem which cannot be easily solved. Other static generation tools look now more promising than 3 years ago, for example [Hugo](https://gohugo.io) or [Grav](https://getgrav.org). 
You can also go full retro and use Wordpress. 👴

Going back to the GatsbyJS rant... I know that the worst is just around the corner. I know that at some point Gatsby will be updated to version 3. I'm more than sure 
there will be plenty of breaking changes about which Gatsby informs in the console log. I hope the migration won't be painful otherwise I will consider Jekyll, Hugo, or Grav 😈.

Jekyll works out-of-the-box, and it can be hosted for free on GitHub which is really great! If you want to create
valuable content and share it with others, then use it. If you want to play with JS and need to do fancy stuff, then Gatsby with GraphQL might be a good choice.

![sketch and jekyll](/assets/2021-01-17/sketch.png)

Checkout [Jekyll Showcase page](https://jekyllrb.com/showcase/). I was personally surprised how many production web pages are built on
top of Jekyll!

This post is too short to have a deeper conclusion but look at the UI/UX design, but did you notice that many software systems 
are now trying more than ever hide its complexity?
