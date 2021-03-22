<template>
  <h1>{{ title }}</h1>
</template>

<script>
import { ref } from '@vue/composition-api';
export default {
  setup() {
    let title = ref('Hello, Vue 3!');

    setTimeout(() => {
      title.value = 'THIS IS A NEW TITLE';
    }, 5000);

    return { title };
  },
};
</script>
