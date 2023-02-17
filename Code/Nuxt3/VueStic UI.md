---
tags: vuestic, nuxt3, frontend, ui-library
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Installation
```ad-important
This section was updated at [[2022-10-05]].

Used with Nuxt 3.
```
References:
- [Vuestic documentation](https://vuestic.dev/en/getting-started/nuxt)

## Steps
- Install `npm install @vuestic/nuxt`
- Modify the `nuxt.config.ts`
```ts
export default defineNuxtConfig({
  modules: ['@vuestic/nuxt'],

  vuestic: {
    config: {
      // Config here
    }
  }
})
```
- Customize colors config
```ts
export default defineNuxtConfig({
  modules: ['@vuestic/nuxt'],

  vuestic: {
    config: {
      colors: {
	      primary: '#fff',
	      ...
      }
    }
  }
})
```

## Misc.
### Override Vuestic Style
- create a new `@/assets/css/main.css` file.
- define the new styles [reference](https://vuestic.dev/en/styles/css-variables#global-overriding).
- import the above file from `app.vue`
```vue
<template></template>
<style>
@import '@/assets/css/main.css';
</style>
```

### Customize Icon
- Install any package of your preferred icon. Example:
	- `npm i -D @unocss/preset-icons @iconify-json/fa6-solid`
	- The above icon set is applied as a [[UnoCSS]] plugin, that can be rendered with `<div class="i-fa6-solid:caret-down" />` element tag.
- Create new Vuestic icon configuration inside the `app.vue`:
	```ts
	<script setup lang="ts">
	const { mergeGlobalConfig } = useGlobalConfig();
	const setNewVuesticConfig = () => {
		mergeGlobalConfig({
			icons: [
				{
					name: 'i-{iconName}',
					resolve: ({ iconName }) => ({
						class: `i-${iconName}`,
						tag: 'div',
						content: null,
					}),
				},
			],
		});
	};
	setNewVuesticConfig();
	</script>
	```
	- References:
		- The above new icons configuration cannot be done inside the `nuxt.config.ts`, because the `dynamicSegments` or `({ iconName })` is can not be accessed. But all of it is possible to do within the `app.vue` fie.
		- [Vuestic global config](https://vuestic.dev/en/services/global-config).
		- [How to create a new icons configuration ](https://vuestic.dev/en/services/icons-config#interactive-playground).