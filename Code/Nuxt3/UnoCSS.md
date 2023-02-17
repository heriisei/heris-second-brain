---
tags: nuxt3, unocss
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Installation
```ad-important
This section was updated at [[2022-10-03]]
```
## Steps
- Install `npm i -D @unocss/nuxt`  [UnoCSS/Nuxt](https://github.com/unocss/unocss/tree/main/packages/nuxt)
- Setup the `nuxt.config.ts`
```ts
export default {
  modules: [
	'@unocss/nuxt',
  ],
  unocss: {
	// presets
	uno: true, // enabled `@unocss/preset-uno`
	// or
	wind: true, // enabled `@unocss/preset-wind`
  },
}
```
- Install `npm i -D @unocss/transformer-variant-group` [Variant Group](https://github.com/unocss/unocss/tree/main/packages/transformer-variant-group).
```ts
import { transformerVariantGroup } from 'unocss';

export default defineNuxtConfig({
	...
	unocss: {
		transformers: [
			transformerVariantGroup()
		],
	},
})
```
- Install `npm i -D @unocss/preset-icons @iconify-json/twemoji`  [Icons](https://github.com/unocss/unocss/tree/main/packages/preset-icons), or any of the [iconset](https://icon-sets.iconify.design/) with the following package name format: `@iconify-json/{iconname}`. Example: from the  `https://icon-sets.iconify.design/fa6-solid/` icon can be translated to : `@iconify-json/fa6-solid`
```ts
export default defineNuxtConfig({
	...
	unocss: {
		...
		icons: true,
		...
	},
})
```
