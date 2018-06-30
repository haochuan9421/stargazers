<template>
  <div>
    <div class='main'>
      <router-view></router-view>
    </div>
    <footer>本页面仅用于交流和学习,请勿滥用信息!!!</footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {

    }
  },
  created () {

  },
  methods: {

  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
html,body{
  width: 100%;
  height: 100%;
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}
.main{
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 70px;
  padding: 20px;
  overflow: auto;
}
footer{
  height: 70px;
  line-height: 70px;
  text-align: center;
  position: fixed;
  left: 0;
  right: 0;
  bottom: 0;
  color: white;
  background-color: rgba(0, 0, 0, .4)
}
</style>
