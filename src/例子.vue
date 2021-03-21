<template>
  <div id="app">
    <danmaku :data="danmaku"
             :src="videoUrl"
             @send="sendDanmaku"
             :width="1200"></danmaku>
  </div>
</template>

<script>
import danmaku from './components/danmaku'

export default {
  name: 'App',
  components: { danmaku },
  data () {
    return {
      danmaku: [],
      videoUrl: 'https://stupid-dragon.oss-cn-beijing.aliyuncs.com/vue-video-danmaku/d.mp4'
    }
  },
  mounted () { this.getDanmaku() },
  methods: {
    getDanmaku () {
      // 请自己实现从服务器获取弹幕
      // 这里只是演示性添加
      this.danmaku = [
        {
          content: 'hello world',
          time: 0,
          type: 'scroll',
          color: '#ffffff'
        },
        {
          content: 'Crzy633制作的danmaku插件',
          time: 0,
          type: 'scroll',
          color: '#ffffff'
        },
        {
          content: '基于vue、element-ui、scss',
          time: 2000,
          type: 'scroll',
          color: '#00afff'
        },
        {
          content: '还使用了阿里矢量库图标',
          time: 2500,
          type: 'scroll',
          color: 'green'
        },
        {
          content: 'https://github.com/Crzy633/vue-video-danmaku',
          time: 7000,
          type: 'top',
          color: '#00afff'
        },
        {
          content: '2021/3/20 01:11',
          time: 5000,
          type: 'bottom',
          color: 'golden'
        }
      ]
    },
    sendDanmaku (danmu) {
      // 请自己实现将弹幕保存到服务器里边
      console.log(danmu)
    }
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
</style>
