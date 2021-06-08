<template>
  <div>
    <p>{{ defaultText }}</p>
    <a @click="openOn" href="https://github.com/kocal/vue-web-extension">uic swagger</a>
    <br>
    <a href="https://www.w3schools.com">uic swagger</a>
    <br>
    <a href="https://www.w3schools.com">airdb.dev</a>
    <br>
    <a href="https://www.w3schools.com">airdb.io</a>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  mounted () {
    browser.runtime.sendMessage({})
  },
  methods: {
    openOn() {
      console.log("xxx")
      chrome.tabs.create( {
	url: "https://github.com/kocal/vue-web-extension"
      })
    },
    openOnNewTab(newTab) {
      console.log(newTab)
      chrome.tabs.create(newTab)
    }
  },
  computed: {
    defaultText () {
      return browser.i18n.getMessage('extName')
    }
  }
}
</script>

<style scoped>
p {
  font-size: 20px;
}
</style>
