<template>
  <div>
    <div>About Child Component</div>
    {{ value }}
  </div>
</template>

<script>
import { toRefs } from '@vue/composition-api';
export default {
  props: {
    title: {
      type: String,
    },
  },
  setup(props) {
    // props 자체는 반응성이 있지만
    // props 내부의 값은 반응성이 없다.
    // 그렇기 때문에 구조분해할당으로 밑에 처럼 빼주게 되면 반응성을 따로 주어야 되므로
    // toRefs안에 넣어준다.
    const { title } = toRefs(props);
    console.log(title);
    return {
      value: title,
    };
  },
};
</script>

<style></style>
