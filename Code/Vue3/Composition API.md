---
tags: vue, vue/vue3, vue/vue3/composition-api, frontend
---

# Props
## Type Definition
### Object
References:
- https://stackoverflow.com/a/64832307

```ts
<script setup lang="ts">
import type { PropType } from 'vue';

interface MyInterface {
	prop1: string;
	...
}

const props = defineProps({
	myObject: {
		required: true,
		type: Object as PropType<MyInterface>
	}
})
</script>
```
