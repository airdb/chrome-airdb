<template>
  <div>
    <p>{{ defaultText }}</p>
    <span @click="goHome" class="home"
      ><i class="iconfont">Home &#xe601;</i></span
    >
    <div class="" @keydown="$event;" @keyup.ctrl="ctrlHandler">
      <div class="">
        <input
          type="text"
          id="textbox"
          value=""
          placeholder="search"
          autofocus
        />
      </div>
    </div>

    <div v-for="(item, index) in items" :key="index">
      <a href="#" @click="openOn(item.url)">{{ item.name }}</a>
    </div>
  </div>
</template>

<script>
export default {
  name: "HelloWorld",
  mounted() {
    browser.runtime.sendMessage({});
    document.getElementById("textbox").focus();
  },
  data() {
    return {
      items: [
        {
          name: "vue-chrome-extersion",
          url: "https://github.com/airdb/chrome-airdb",
        },
        { name: "uic swagger", url: "https://scf.baobeihuijia.com/test/uic/" },
        { name: "login", url: "https://scf.baobeihuijia.com/test/uic/login" },
        { name: "airdb.dev", url: "https://airdb.dev" },
        { name: "airdb.io", url: "https://airdb.io" },
        { name: "extention", url: "/index.html" },
        { name: "chrome-dns", url: "chrome://net-internals/#dns" },
      ],
    };
  },
  methods: {
    focusInput() {
      console.log("bbb");
      this.$refs.search.$el.focus;
      console.log("ccc");
    },
    goHome() {
      chrome.tabs.create({
        url: "https://noah.airdb.io",
        selected: true,
      });
    },
    openOn(url) {
      console.log("xxx");
      chrome.tabs.create({
        url: url,
      });
    },
    openOnNewTab(newTab) {
      console.log(newTab);
      chrome.tabs.create(newTab);
    },
  },
  computed: {
    defaultText() {
      return browser.i18n.getMessage("extName");
    },
  },
};
</script>

<style scoped>
p {
  font-size: 20px;
}

.home {
  width: 30px;
  height: 30px;
  font-size: 20px;
  cursor: pointer;
}
</style>
