<template>
  <div id="app">
    <div id="box">
      <scrollbar @bottom="bottom" @top="top" @left="left" @right="right" :changeLeft.sync="changeLeft" :changeTop.sync="changeTop">
        <div id="container">
            <img src="./assets/logo.png">
            <p>wecome to me github</p>
            <p><a href="https://github.com/toMatthew/vue-scrollbar">https://github.com/toMatthew/vue-scrollbar</a></p>
            <p v-for="n in 20">vue自定义滚动条---{{n}}</p>
            <div v-for="(str,index) in data">add data --- {{str}} --- {{index}}</div>
        </div>
      </scrollbar>
      <div id="loadingbox" v-show="isloading">loading...</div>
    </div>
  </div>
</template>

<script>
import scrollbar from './components/vue-scrollbar'

export default {
  name: 'app',
  components: {
    scrollbar
  },
  data () {
    return {
        data:[],
        isloading : false,
        changeLeft : 50,
        changeTop : 100,
    }
  },
  methods:{
    top() {
      console.log('top');
    },
    bottom() {
      console.log('bottom')
      this.isloading = true;
      var newdata = [
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
          '我是新加的数据',
      ];
      setTimeout(()=>{
        this.data = this.data.concat(newdata);
        this.isloading = false;
      }, 1000);
    },
    left() {
      console.log('left')
    },
    right() {
      console.log('right')
    }
  },
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
#box{
  margin: 0 auto;
  width: 300px;
  height: 500px;
  border:1px solid #eee;
  border-radius: 3px;
  position: relative;
}
#container{
  width: 400px;
}
#loadingbox{
  width: 100%;
  background: rgba(255,255,255,.8);
  position: absolute;
  left: 0;
  bottom: 0;
  text-align: center;
  font-weight: bold;
}
</style>
