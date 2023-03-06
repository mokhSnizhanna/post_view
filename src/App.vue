<template>
  <div id="post">
    <post-view msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
import PostView from './components/PostView.vue'

export default {
  name: 'App',
  components: {
    PostView
  },
  data() {

  }
}
</script>

<style>
#app {

}
</style>
