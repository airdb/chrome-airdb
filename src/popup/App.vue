<template>
  <hello-world />
</template>

<script>
import HelloWorld from '@/components/HelloWorld.vue'
import dashbord from '@/components/dashbord.vue'

export default {
  name: 'App',
  components: { HelloWorld }
}
</script>

<style>
html {
  width: 400px;
  height: 400px;
}
</style>
