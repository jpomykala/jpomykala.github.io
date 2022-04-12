---
layout: post 
title: "🔎 How I integrated Algolia search with NextJS and markdown files?"
tags: ['nextjs', 'devlog']
image: https://jpomykala.com/assets/2022-04-12/example-search-results.png
---


Since 2019 I'm building a [SaaS for translation management](https://simplelocalize.io) as a solo developer. Since the very beginning,
I needed a place where I could post code samples, and explain people how to integrate different frameworks or how
to use SimpleLocalize CLI to upload and download translation files. 
You can see Algolia search in action on [SimpleLocalize documentation](https://simplelocalize.io/docs/).


![algolia nextjs markdown example](/assets/2022-04-12/example-search-results.png)

At beginning, I started using [GitBook](https://www.gitbook.com) software because it was shining and trendy at the time. It was a good move,
because when you start something new you don't want to start from scratch on every level. The main goal is to deliver good quality code and value to the customers, 
all other things you can (or should) buy to save time. 

Short story long, I wasn't quite happy with the GitBook after couple of months, because it didn't fit to my needs, I didn't like
the style, and I was spending too much time on adjusting and fixing links and so on. I decided to go with my custom documentation, and 
surprisingly everything went smoothly! The only concert was lack o good search ability which is available in all documentation solutions, 
or [open-source Docusaurus](https://github.com/facebook/docusaurus).


## Installation

Algolia search components and Algolia client installation
```shell
npm install react-instantsearch-dom algoliasearch --save
```

[gray-matter](https://github.com/jonschlinkert/gray-matter) a Markdown parser and [globby](https://github.com/sindresorhus/globby) for finding files
```shell
npm install gray-matter globby --saveDev
```

## Configuration
```typescript
import algoliasearch from 'algoliasearch/lite';
import {connectStateResults, Hits, InstantSearch, SearchBox} from 'react-instantsearch-dom';

const searchClient = algoliasearch(
  'APP_ID',
  'SEARCH_API_KEY'
);
```

### Results component

By default, Algolia `<Hits/>` component shows all results when search query is empty. I adjusted the code to show results only if there are any results and data are loaded.

```tsx
const Results = connectStateResults(({searchState, searchResults, searching}) => {
  const hasQuery = searchState && searchState.query;
  const hasResults = (searchResults?.hits ?? []).length > 0;
  const isSearching = searching;
  if (hasQuery && hasResults) {
    return <Hits hitComponent={Hit}/>;
  }
  if (hasQuery && !hasResults && !isSearching) {
    return <div>No results 😔</div>
  }
  return null;
}
);
```


### Hit component

Hit component is nothing else as just a one search result. I stripped my styling to make it easier to copy-paste. 😄 

```tsx
import React from "react";

function Hit(props: any) {
  const content = props?.hit?.content ?? "";
  const words = content.split(" ").length;
  return (<a href={props.hit.slug}>
    <div>
      <h3>{props?.hit?.frontmatter?.title ?? "no title"}</h3>
    </div>
    <p>{props?.hit?.frontmatter?.excerpt ?? ""}</p>
  </a>)
}

export default Hit;
```

### Search component

Use `InstantSearch` component and search client which you configured in the previous step, put your index name, and 
use Algolia `SearchBox` component and our custom `Results` component with changed empty state behaviour which we created previously.

```tsx
<InstantSearch indexName="simplelocalize-docs" searchClient={searchClient}>
    <SearchBox/>
    <Results/>
</InstantSearch>
```
I didn't want to make it more complex that it needs to be, I overwrite some CSS properties in my stylesheets to hide a search button and reset button. 
I also added Bootstrap styling to the search box using SCSS `@extend` property. Simple and it does the job.

```scss
.ais-SearchBox-input {
  @extend .form-control;
}

.ais-SearchBox-submit {
  display: none;
}

.ais-SearchBox-reset {
  display: none;
}
```


## Get markdown files to index

Now, it's time to get all files which should be indexed and visible in the search results.
I created a new `index-docs.js` file which I will be executed after every successful build on CI/CD server.
All my documentation pages are in `/docs/` directory. So I needed to write a script for converting it into an array of objects to populate Algolia search index.

```typescript
import {globby} from 'globby';
import fs from "fs";
import matter from "gray-matter";
import algoliasearch from 'algoliasearch';

const pages = await globby([
  'docs/',
]);

const objects = pages.map(page => {
  const fileContents = fs.readFileSync(page, 'utf8')
  const {data, content} = matter(fileContents)
  const path = page.replace('.md', '');
  let slug = path === 'docs/index' ? 'docs' : path;
  slug = "/" + slug + "/"
  return {
    slug,
    content,
    frontmatter: {
      ...data
    }
  }
})

//algolia update index code
```

- I'm getting all markdown files from `/docs/` directory with `globby`, 
- reading content of the files with `fs`,
- parsing files with `gray-matter` for markdown, 
- and doing some magic tricks to convert file path to slugs which are used.

In my case, slugs are the just file paths, where the file path looks like this `/docs/{category}/{title}.md`, for example:
`/docs/integrations/next-translate.md`, so slug will look like this: `/docs/integrations/next-translate/`

In the end I'm getting an array of objects which looks like this:

```json
[
    {
      "slug": "/docs/integrations/next-translate/",
      "content": "My article content for indexing",
      "frontmatter": {
        "category": "Integrations",
        "date": "2022-01-20",
        "some-other-properties": "properties"
      }
    }
]
```

### Update Algolia index

Algolia provides a really simple and easy to use JavaScript client, so there no magic here. We are acquiring our index, and
saving objects which we created in previous step.

```typescript
const client = algoliasearch(
  'APP_ID',
  'ADMIN_API_KEY'
);
const index = client.initIndex("simplelocalize-docs")
index.saveObjects(objects, {
  autoGenerateObjectIDIfNotExist: true
});
```


### Include index update in package.json

```json
{
  "name": "simplelocalize-docs",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build && next export -o build/ && i18n:download && index:docs",
    "index:docs": "npx ts-node --skip-project index-docs.mjs",
    "i18n:upload": "simplelocalize upload",
    "i18n:download": "simplelocalize download"
  }
}
```

You can execute the script manually by running `npm run index:docs`.

![algolia search index overview](/assets/2022-04-12/algolia-docs-search-index-overview.png)

That's it? That's it! See the search in action on SimpleLocalize [documentation page](https://simplelocalize.io/docs).

[//]: # (<div style="padding:100% 0 0 0;position:relative;"><iframe src="https://player.vimeo.com/video/698425700?h=d3c47c4475&amp;badge=0&amp;autopause=0&amp;player_id=0&amp;app_id=58479" frameborder="0" allow="autoplay; fullscreen; picture-in-picture" allowfullscreen style="position:absolute;top:0;left:0;width:100%;height:100%;" title="algolia-search-with-markdown-and-next-js-example"></iframe></div><script src="https://player.vimeo.com/api/player.js"></script>)
