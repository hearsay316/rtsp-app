<template>
  <div id="app">
    <div id="nav">
      <router-link to="/">Home</router-link> |
      <router-link to="/about">About</router-link> |
      <router-link to="/rtsp">Rtsp</router-link>
    </div>
    <router-view />
  </div>
</template>

<style lang="stylus">
body,html{
  margin 0
  padding 0
}
#app
  font-family Avenir, Helvetica, Arial, sans-serif
  -webkit-font-smoothing antialiased
  -moz-osx-font-smoothing grayscale
  color #2c3e50
  margin-top 60px
#nav
  text-align center
</style>
