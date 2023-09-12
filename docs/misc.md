# Miscellaneous

A catch-all category for everything else.

## Tailwind, Sveltekit and PostCSS

In order to get these all to play nice, I had to add the following to my `postcss.config.cjs` file:

```js
const path = require('path');
const tailwindcss = require('tailwindcss');
const autoprefixer = require('autoprefixer');

module.exports = {
	plugins: [tailwindcss(path.resolve(__dirname, './tailwind.config.ts')), autoprefixer]
};
```
