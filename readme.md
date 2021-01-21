
<p align="center">
  <a href="https://sphido.org">
    <img src="https://sphido.org/img/sphido.svg" width=""/>
  </a>
</p>


<p align="center">
  A rocket 🚀 fast, lightweight, static site generator.
</p>

## Installation

```bash
$ npm i @sphido/corecore @sphido/nunjucks @sphido/frontmatter @sphido/markdown @sphido/meta
```

## Quick Start

```javascript
import globby from 'globby';

import {save} from '@sphido/nunjucks';
import {frontmatter} from '@sphido/frontmatter';
import {meta} from '@sphido/meta';
import {markdown} from '@sphido/markdown';
import {getPages} from '@sphido/core';



(async () => {
	// 1. get list of pages...
	const posts = await getPages(
		await globby('packages/**/*.md'),
		...[
			frontmatter,
			markdown,
			meta,
			{save}
		]
	);

	// 2. save to html with default template
	for await (const page of posts) {
		await page.save(
			page.dir.replace('packages', 'public'),
		);
	}
})();
```

## Packages

```bash
npm i @sphido/core           # basic getPages, getPage functions
npm i @sphido/emoji          # add twemoji support
npm i @sphido/feed           # generate atom feed from pages
npm i @sphido/frontmatter    # frontmatter for pages
npm i @sphido/link           # add link() method to pages
npm i @sphido/markdown       # markdown page processor 
npm i @sphido/meta           # add common metadata to the pages
npm i @sphido/nunjucks       # nunjucks templates
npm i @sphido/pagination     # paginate over pages
npm i @sphido/sitemap        # generate sitemap.xml
```

## Examples

* https://github.com/sphido/examples/

## License

MIT