<template>
  <div class="about">
    <h1>This is an about page</h1>
    <about-child-component :title="title" />
  </div>
</template>

<script>
import AboutChildComponent from './AboutChildComponent.vue';
import { onMounted, ref } from '@vue/composition-api';
export default {
  components: { AboutChildComponent },
  setup() {
    const title = ref('yugesh');

    onMounted(() => {
      setTimeout(() => {
        title.value = 'dngwoo';
      }, 5000);
    });

    return {
      title,
    };
  },
};
</script>
