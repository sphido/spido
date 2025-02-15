# @sphido/core

Sphido core package contains two most important function `getPages()` and `allPages()`. The `getPages()` function scans
directories for all `*.md` and `*.html` files. Second function `allPages()`
is [generator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator)
that allow to iterate over all pages.

```javascript
const pages = await getPages({path: 'content'}, /* ...exteners */);
```

Returned structure is very simple and looks like follow:

```json
[
	{
		"name": "Main page",
		"path": "content/Main page.md"
	},
	{
		"name": "Directory",
		"children": [
			{
				"name": "Subpage one",
				"path": "content/Directory/Subpage one.md"
			},
			{
				"name": "Subpage two",
				"path": "content/Directory/Subpage two.md"
			}
		]
	}
]
```

Then iterate over pages like follow:

```javascript
for (const page of allPages(pages)) {
	console.log(page);
}
```

## Extending `page` object

Every single `page` object inside structure can be modified with extender. Extenders are set as additional parameters of
the `getPages()` function. There are two types of extenders:

### *Callback* extenders

Callback extender is a function that is called during recursion over each page with three parameters passed to the
function `page`, `path` and [`dirent`](https://nodejs.org/api/fs.html#class-fsdirent).

```javascript
const callbackExtender = (page, path, dirent) => {
	// do anything with page object
}

const pages = await getPages({path: 'content'}, callbackExtender);
```

### *Object* extenders

This extender is just a simple JavaScript object that is combined with the `page` object using
the  [Object.assign()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
function.

```javascript
const objectExtender = {
	author: 'Roman Ožana'
}

const pages = await getPages({path: 'content'}, objectExtender);
```

There is no limit to the number of extenders, you can combine as many as you want. Let's have the following code:

```javascript
const extenders = [

	// callback extenders will be called during iteration ony by one

	(page) => {
		// add property
		page.title = `${page.name} | my best website`;

		// or function
		page.getDate = () => new Date();

		// or something else 
		page.counter = 1;
	},

	// callback extenders are called in the series

	(page) => {
		page.counter++;
	},

	// object extender will be merged with page object

	{
		"author": "Roman Ožana",
		"getLink": function () {
			return this.path.replace('content', 'public');
		}
	}
];

const pages = getPages({path: 'content'}, ...extenders);
```

then you get this structure:

```json
[
	{
		"name": "Main page",
		"path": "content/Main page.md",
		"title": "Main page | my best website",
		"counter": 2,
		"author": "Roman Ožana",
		"getDate": "[Function: getDate]",
		"getLink": "[Function: getLink]"
	}
]
```

## Installation

```bash
yarn add @sphido/core
```

## Example

Following example read all `*.md` files in `content` directory and process them
with [marked](https://github.com/markedjs/marked) to HTML files

```javascript
#!/usr/bin/env node

import { dirname, relative, join } from 'node:path';
import { getPages, allPages, readFile, writeFile } from '@sphido/core';
import slugify from '@sindresorhus/slugify';
import { marked } from 'marked';

const pages = await getPages({path: 'content'}, // ... extenders
	(page) => {
		page.slug = slugify(page.name) + '.html';
		page.dir = dirname(page.path);
	});

for (const page of allPages(pages)) {
	page.output = join('public', relative('content', page.dir), page.slug);
	page.content = marked(await readFile(page.path));

	await writeFile(page.output, `<!DOCTYPE html>
		<html lang="en" dir="ltr">
		<head>
			<meta charset="UTF-8">
			<script src="https://cdn.tailwindcss.com?plugins=typography"></script>
			<title>${page.name} | Sphido Example</title>
		</head>
		<body class="prose mx-auto my-6">${page.content}</body>
		<!-- Generated by Sphido from ${page.path} -->
		</html>
  `);
}
```

## Source codes

[@sphido/core](https://github.com/sphido/sphido/tree/main/packages/sphido-core)