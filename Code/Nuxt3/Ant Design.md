---
tags: Nuxt3, AntDesign
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Installation
## Steps
- Install `npm install ant-design-vue` 
- Create `plugins/antd.ts`
```ts
import 'ant-design-vue/dist/antd.css';
import Antd from 'ant-design-vue';

export default defineNuxtPlugin((nuxtApp) => {
	nuxtApp.vueApp.use(Antd)
});
```
