---
tags: quasar, vue, frontend
---

# Init
## - VS Code, Linting, Formatter
### -- Auto formatter
#auto-formatter-quasar
- Install `eslint-config-prettier`
	- reference: https://eslint.vuejs.org/user-guide/#conflict-with-prettier
	- `npm install --save-dev eslint-config-prettier`
- `.eslintrc.js`
	```javascript
	extends: [
		'plugin:vue/vue3-recommended',
		'airbnb-base',
		// Make sure that prettier is the last in the order
		'prettier',
	],
	rules: { // Optional
		quotes: ['error', 'single', { avoidEscape: true }],
		'vue/no-template-target-blank': [
			'error',
			{
				allowReferrer: true,
				enforceDynamicLinks: 'always',
			},
		],
	}
	```
- `.prettierrc.js`
	```javascript
	// Matching the singlequote as ESLint do
	module.exports = {
		singleQuote: true,
		// semi: false,
		trailingComma: "es5",
	};
	```
- `.vscode/settings.json`
	- `"editor.defaultFormatter": "esbenp.prettier-vscode",`
		- reference: https://quasar.dev/start/vs-code-configuration#vite-and-vue-cli-and-umd
	- `"eslint.validate": []` add `"html"`

### -- Snippet
- Form Components internal validation
	- Rules for numbers (year)
	```ts
	const rules = ref({
		year: [
			(val: string) =>
				(val && val.length > 0) ||
				t('validate.mustFilled', { object: t('general.paperYear') }),
			(val: string) =>
				(/^(\d)*$/.test(val) && val.length === 4) ||
				t('validate.mustCorrectFormat', {
					object: t('general.year'),
					example: '2022',
				}),
		],
	});
	```