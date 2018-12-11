<template>
  <div id="app">
    <input-box :store="list" filter="^[\w-]*$" placeholder="Add Item" />
  </div>
</template>

<script>
import InputBox from "./components/InputBox.vue";

export default {
  name: 'app',
  data() {
    return {
      list: [
        'test-1',
        'test-2',
        'test-3'
      ].map(i => ({ value: i })),
    }
  },
  components: {
    InputBox
  }
}
</script>

<style lang="scss">
#app {

}
</style>
