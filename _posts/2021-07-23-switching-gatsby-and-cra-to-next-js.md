---
layout: post 
title: "💬 How I switched Gatsby and CRA to NextJS"
tags: ['gatsby', 'cra', 'react', 'nextjs', 'js to ts']
---

When I started working on [SimpleLocalize](https://simplelocalize.io) I started with a very simple PoC app built with `create-react-app`. 
Later when the project grew I used GatsbyJS to build landing page. Then I converted documentation written in [GitBook](https://www.gitbook.com/) to [GatsbyJS](https://www.gatsbyjs.com/) 
documentation page with a markdown. Why I'm doing this? 

### Pros
- one codebase
- shared styles
- shared components
- less dependency issues
- small, consistent and powerful routing API
- feels more mature and stable
- faster feature development due to **fast reloading**

### Cons:
- project could become a big ball of mud


## Why I didn't just choose Gatsby instead NextJS?

![gatsby sharp error](https://jpomykala.com/assets/2021-03-31/gatsby-error.png)

I was tired with GraphQL and fixing problem with Gatsby. 
Most of the time which I spent with Gatsby, was fixing issues or errors thrown by plugins or Gatsby. Some of my pages
are really complex, and they provide custom i18n support. In some cases production builds does not match
to the dev ones. CSS are different or page jumps due to duplicated HTML tags. I spent huge amount of time on configuring Gatsby.

![gatsby new features](https://jpomykala.com/assets/2021-03-31/gatsby-new-features.png)

My Gatsby config has been changed over-the-air so if I want I can disable it. 
Why somebody is changing my working setup? 🤨 


## Migration process

I started with the Migration guide on NextJS docs page.


My wrong assumptions. For the whole time of NextJS existence I thought i needs a NodeJS server, because it's SSR. 


I decided to move SimpleLocalize from CRA to NextJS and also move my SimpleLocalize landing page built with Gatsby to NextJS and maintain only one source code instead 2. 

I started with the [CRA migration guide](https://nextjs.org/docs/migrating/from-create-react-app), but it does not cover everything.


### Migrating CRA Redux to NextJS
In my app, I'm using HOC wrappers everywhere like below:
```
export default connect(mapStateToProps, mapDispatchToProps)(LoginForm);
```

I tried to adjust my code according to example from [vercel/next.js/with-redux repository](https://github.com/vercel/next.js/tree/canary/examples/with-redux), but it didn't work. 
I [found on StackOverflow that this way of integrating redux is obsolete](https://stackoverflow.com/questions/62761838/uncaught-typeerror-store-getstate-is-not-a-function-nextjs-app), 
and I should go with [next-redux-wrapper](https://github.com/kirill-konshin/next-redux-wrapper).

I copied the example from the next-redux-wrapper repo.
```tsx
// store.ts

import {createStore, AnyAction} from 'redux';
import {MakeStore, createWrapper, Context, HYDRATE} from 'next-redux-wrapper';

//Here I used my CRA reducer
const reducer = {};

// create a makeStore function
const makeStore: MakeStore<State> = (context: Context) => createStore(reducer);

// export an assembled wrapper
export const wrapper = createWrapper<State>(makeStore, {debug: true});
```

I only adjusted it to my currently used redux setup and that's it. Works like a charm!

### Migrating images from GatsbyJS and CRA to NextJS

Usually in GatsbyJS and CRA you load images like this.
```tsx
import myImage from "../assets/my-image.jpg";
```

```html
<img src={myImage}/>
```

In NextJS images are always loaded from the `public` folder. 
I replaced `import` statements with `const` and change the `from` part into assigment like below:
```tsx
const myImage = "../assets/my-image.jpg"
```

Now, all images are just `const` variables, and `<img/>` tags stayed as it is.


### Migrating JS files to TS files and JSX to TSX


#### JS to TS

```
find src/ -name "*.js" -exec sh -c 'mv "$0" "${0%.js}.ts"' {} \;
```

#### JSX to TSX

```
find src/ -name "*.jsx" -exec sh -c 'mv "$0" "${0%.jsx}.tsx"' {} \;
```
