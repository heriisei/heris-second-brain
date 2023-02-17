---
tags: nuxt3, frontend, storybook
---

```ad-example
title: Navigation
👈 Previous - [[]]

👉 Next - [[]]
```

# Initialization
## Install storybook
References:
- [Documentation](https://storybook.js.org/docs/vue/get-started/install).
- [Storybook for vite](https://storybook.js.org/blog/storybook-for-vite/).

### Steps
- Install storybook with `npx storybook init --type vue3 --builder @storybook/builder-vite`
```ad-info
Note: If you are using a version of npm above npm7, Storybook will ask if you would like to run the npm7 migration. Press ‘y’ to answer yes to run the migration. If you do not do this, you will have issues installing the other packages needed to continue this set up. If you skip this step by accident, run `npx storybook@next automigrate` to run the migration.
```
- Run it by `npm run storybook`

