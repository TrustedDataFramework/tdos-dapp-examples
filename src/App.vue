<template>
  <div id="examples-app">
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>
<style>
   @import url('assets/fonts/SourceHanSansCN-Light.css');
</style>
<style lang="less">

#examples-app {
  font-family: 'SourceHanSansCN-Light','Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
 width:100%;height:100%;

}
input,textarea{font-family:'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;}



</style>
