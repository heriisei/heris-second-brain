---
tags: vue3, vitesse, histoire
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Initialization
## Initialize the project
- Generate a new project via https://github.com/antfu/vitesse/generate.
- Clone the github repo into the local computer
- Edit the `.npmrc` file into like this:
	```.npmrc
	/* .npmrc */
	   shamefully-hoist=true
	   strict-peer-dependencies=false
	++ auto-install-peers=true
	```
	to solve problems with Histoire, that stated from [Akryum](https://discord.com/channels/1007690324329648168/1007693771128971355/1031202373529505812) and  [Vue.js Forum](https://forum.vuejs.org/t/vue3-onmounted-function-doesnt-work-in-esm-package-bundled-with-rollup/124471), then I found and try this command [pnpm link](https://pnpm.io/cli/link#pnpm-link-dir).
	![Vitesse error with Histoire integration](vitesse-histoire-link.png)
- Run `pnpm i` to install the dependencies.
- Run `pnpm dev` to start the dev server.

## Integrate with Histoire
- Install with `pnpm i -D histoire @histoire/plugin-vue`.
- Add these scripts into `package.json`
```.json
// package.json
{
	"scripts": {
		"story:dev": "histoire dev",
		"story:build": "histoire build",
		"story:preview": "histoire preview"
	}
}
```
- Create a `histoire.config.ts` following the [Docs](https://histoire.dev/guide/vue3/stories.html).
```ts
// histoire.config.ts
import { defineConfig } from 'histoire'
import { HstVue } from '@histoire/plugin-vue'

export default defineConfig({
	plugins: [
		HstVue(),
	],
	setupFile: '/src/histoire.setup.ts',
})
```
- Create a `/src/histoire.setup.ts` global setup, following the [Docs](https://histoire.dev/guide/vue3/stories.html).
```ts
// /src/histoire.setup.ts
import { defineSetupVue3 } from '@histoire/plugin-vue'
import 'uno.css'

export const setupVue3 = defineSetupVue3(({ app }) => {
	app.use(...)
})
```
- Create a `YourComponent.story.vue` following the [Docs](https://histoire.dev/guide/vue3/stories.html).
- Run `pnpm story:dev` to start the Histoire dev server.

## Integrate with Vuestic
- Install with `pnpm add vuestic-ui`, [Docs](https://vuestic.dev/en/getting-started/installation#manual-installation).
- Setup with tree shaking configuration, following the [Docs](https://vuestic.dev/en/getting-started/tree-shaking).
```ts
// /src/main.ts
...
import {
	VaButton, VaInput, VaSelect,
	createVuesticEssential,
} from 'vuestic-ui'
import 'vuestic-ui/styles/essential.css'
import 'vuestic-ui/styles/typography.css'

// https://github.com/antfu/vite-ssg
export const createApp = ViteSSG(
	...
	(ctx) => {
		// install all modules under `modules/`
		...
		ctx.app.use(createVuesticEssential({
			components: { VaButton, VaSelect, VaInput },
		}))
	},
)
```
```ts
// histoire.setup.ts
...
import {
	VaButton, VaInput, VaSelect,
	createVuesticEssential,
} from 'vuestic-ui'
import 'vuestic-ui/styles/essential.css'
import 'vuestic-ui/styles/typography.css'
...

export const setupVue3 = defineSetupVue3(({ app }) => {
	app.use(createVuesticEssential({
		components: { VaButton, VaSelect, VaInput },
	}))
})
```