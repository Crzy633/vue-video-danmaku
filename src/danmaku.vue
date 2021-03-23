<template>
  <div id="danmaku"
       :class="[{ full: controls.full }, { smallScreen }, { tinyScreen }]"
       :style="danmakuStyle">
    <!-- 主容器，全屏作用的对象 -->
    <main class="danmaku-main">
      <!-- 视频 -->
      <video class="danmaku-video"
             :src="src"></video>
      <!-- 容器：绘制弹幕 -->
      <div :class="['danmaku-danmu-container'].concat(setting.block)"
           id="danmaku-danmu-container"
           v-if="setting.onOff">
        <div v-for="danmu in list"
             :id="danmu.id"
             :key="danmu.id"
             :class="['danmaku-danmu', danmu.type]"
             :style="'color:' + danmu.color +
                   ';opacity:' + setting.opacity/100 +
                   ';font-size:' + (setting.size - (smallScreen ? 8 : 0)) + 'px' +
                   ';animation: danmaku' + danmu.type + ' linear ' + (((100 - setting.speed) / 5) - (controls.rate - 4) * 0.75) + 's 1 forwards' +
                   ';animation-play-state: ' + aniState +
                   ';visibility: hidden'">{{danmu.content}}</div>
      </div>
      <!-- 控件 -->
      <div class="danmaku-controls-container">
        <div class="dammaku-controls"
             @mousemove="controls.show = 3">
          <!-- 进度条控件 -->
          <div class="danmaku-progress-container">
            <div class="danmaku-progress">
              <el-slider :class="[
                         'danmaku-progress-slider', 
                         { 'danmaku-progress-hidden': controls.show <= 0 }
                       ]"
                         v-model="controls.dragTime"
                         :max="controls.durationTime"
                         :format-tooltip="formatDragTime"
                         @change="dragEng"></el-slider>
            </div>
          </div>
          <!-- 左浮动控件 -->
          <div class="danmaku-float-left">
            <!-- 播放/暂停按钮控件 -->
            <button class="danmaku-button"
                    @click="playOrPause">
              <i v-if="!isPlaying"
                 class="el-icon-video-play"></i>
              <i v-if="isPlaying"
                 class="el-icon-video-pause"></i>
            </button>
            <!-- 播放时间/视频时长控件 -->
            <span class="danmaku-time">
              <span>{{controls.time1}}/{{controls.time2}}</span>
            </span>
          </div>
          <!-- 弹幕开关控件 -->
          <el-switch class="danmaku-switch"
                     v-model="setting.onOff"></el-switch>
          <!-- 弹幕选项控件 -->
          <span class="danmaku-tooltip danmaku-setting">
            <i class="el-icon-c-scale-to-original"></i>
            <div class="danmaku-tooltip-content">
              <!-- 弹幕选项-屏蔽控件 -->
              <div>按类型屏蔽</div>
              <div class="danmaku-block">
                <button :class="['danmaku-block-button', { active: setting.block.indexOf('scroll') !== -1 }]"
                        @click="blockType('scroll')">
                  <svg class="icon"
                       aria-hidden="true">
                    <use xlink:href="#icon-bofangqi-danmugundongkai"></use>
                  </svg>
                  <br>
                  <span class="danmaku-block-button-explain">滚动</span>
                </button>
                <button :class="['danmaku-block-button', { active: setting.block.indexOf('top') !== -1 }]"
                        @click="blockType('top')">
                  <svg class="icon"
                       aria-hidden="true">
                    <use xlink:href="#icon-bofangqi-danmudingbukai"></use>
                  </svg>
                  <br>
                  <span class="block-explain">顶部</span>
                </button>
                <button :class="['danmaku-block-button', { active: setting.block.indexOf('bottom') !== -1 }]"
                        @click="blockType('bottom')">
                  <svg class="icon"
                       aria-hidden="true">
                    <use xlink:href="#icon-bofangqi-danmudibukai"></use>
                  </svg>
                  <br>
                  <span class="block-explain">底部</span>
                </button>
              </div>
              <!-- 弹幕选项-透明度控件 -->
              <div class="danmaku-line">
                <span class="danmaku-line-label">不透明度</span>
                <el-slider class="danmaku-line-slider"
                           :show-tooltip="false"
                           v-model="setting.opacity"></el-slider>
              </div>
              <!-- 弹幕选项-区域控件 -->
              <div class="danmaku-line danmaku-line2">
                <span class="danmaku-line-label">显示区域</span>
                <el-slider class="danmaku-line-slider"
                           :max="3"
                           :show-stops="true"
                           :show-tooltip="false"
                           :marks="setting.marks"
                           v-model="setting.zone"></el-slider>
              </div>
              <!-- 弹幕选项-速度控件 -->
              <div class="danmaku-line">
                <span class="danmaku-line-label">弹幕速度</span>
                <el-slider class="danmaku-line-slider"
                           :min="1"
                           :max="100"
                           :show-tooltip="false"
                           v-model="setting.speed"></el-slider>
              </div>
              <!-- 弹幕选项-字体控件 -->
              <div class="danmaku-line">
                <span class="danmaku-line-label">字体大小</span>
                <el-slider class="danmaku-line-slider"
                           :min="12"
                           :max="48"
                           :show-tooltip="false"
                           v-model="setting.size"></el-slider>
              </div>
            </div>
          </span>
          <!-- 弹幕输入栏控件 -->
          <el-input v-model="input"
                    class="danmaku-input"
                    :maxlength="50"
                    placeholder="发个弹幕~"
                    @keyup.enter.native="send">
            <!-- 弹幕输入栏选项控件 -->
            <span class="danmaku-tooltip danmaku-input-setting"
                  slot="prepend">
              <i class="el-icon-s-open"></i>
              <div class="danmaku-tooltip-content">
                <!-- 弹幕输入栏选项-类型控件 -->
                <div>类型</div>
                <div class="danmaku-input-setting-block">
                  <button :class="['danmaku-block-button', { active: type==='scroll' }]"
                          @click="sendType('scroll')">
                    <svg class="icon"
                         aria-hidden="true">
                      <use xlink:href="#icon-bofangqi-danmugundongkai"></use>
                    </svg>
                    <br>
                    <span class="danmaku-block-explain">滚动</span>
                  </button>
                  <button :class="['danmaku-block-button', { active: type==='top' }]"
                          @click="sendType('top')">
                    <svg class="icon"
                         aria-hidden="true">
                      <use xlink:href="#icon-bofangqi-danmudingbukai"></use>
                    </svg>
                    <br>
                    <span class="danmaku-block-explain">顶部</span>
                  </button>
                  <button :class="['danmaku-block-button', { active: type==='bottom' }]"
                          @click="sendType('bottom')">
                    <svg class="icon"
                         aria-hidden="true">
                      <use xlink:href="#icon-bofangqi-danmudibukai"></use>
                    </svg>
                    <br>
                    <span class="danmaku-block-explain">底部</span>
                  </button>
                </div>
                <!-- 弹幕输入栏选项-颜色控件 -->
                <div class="danmaku-input-setting-line">
                  <span class="danmaku-input-setting-label">颜色</span>
                  <button @click="chooseColor('#ffffff')"
                          :class="['danmaku-input-setting-color', {active: color==='#ffffff'}]"></button>
                  <button @click="chooseColor('#00afff')"
                          :class="['danmaku-input-setting-color', 'blue', {active: color==='#00afff'}]"></button>
                  <button @click="chooseColor('#ff0000')"
                          :class="['danmaku-input-setting-color', 'red', {active: color==='#ff0000'}]"></button>
                  <button @click="chooseColor('#ffd700')"
                          :class="['danmaku-input-setting-color', 'yellow', {active: color==='#ffd700'}]"></button>
                  <button @click="chooseColor('#008000')"
                          :class="['danmaku-input-setting-color', 'green', {active: color==='#008000'}]"></button>
                </div>
              </div>
            </span>
            <!-- 弹幕输入栏-发送按钮控件-->
            <el-button class="danmaku-input-send"
                       @click="send"
                       slot="append">发送</el-button>
          </el-input>
          <!-- 信息控件 -->
          <span class="danmaku-info"
                v-if="!hideInfo">
            <span v-if="data.length > 0">{{data.length}}条弹幕</span>
          </span>
          <!-- 右浮动控件 -->
          <div class="danmaku-float-right">
            <!-- 倍速控件 -->
            <span class="danmaku-tooltip danmaku-rate">
              <div class="danmaku-tooltip-content">
                <el-slider v-model="controls.rate"
                           class="volumeSlider"
                           :min="1"
                           :max="12"
                           vertical
                           :show-tooltip="false"
                           height="130px"></el-slider>
              </div>
              <span class="danmaku-tooltip-span">{{formatRate}}</span>
            </span>
            <!-- 音量控件 -->
            <span class="danmaku-tooltip danmaku-volume">
              <div class="danmaku-tooltip-content">
                <el-slider v-model="controls.volume"
                           class="volumeSlider"
                           vertical
                           height="80px"
                           :show-tooltip="false"></el-slider>
              </div>
              <svg t="1616225813843"
                   class="icon"
                   viewBox="0 0 1393 1024"
                   version="1.1"
                   xmlns="http://www.w3.org/2000/svg"
                   p-id="2324"
                   width="30"
                   height="30">
                <path d="M942.89302516 765.94440258a38.50918593 38.50918593 0 0 1-27.58090338-11.44867659 39.02957997 39.02957997 0 0 1 0-55.16180677 287.77810526 287.77810526 0 0 0 0-406.94842366 39.02957997 39.02957997 0 1 1 55.16180676-55.16180677 364.27608233 364.27608233 0 0 1 0 517.27203721 36.94800227 36.94800227 0 0 1-27.58090338 11.44867658z"
                      fill="#ffffff"
                      p-id="2325"></path>
                <path d="M835.69177816 658.74315558a41.11115768 41.11115768 0 0 1-27.58090337-11.44867658 39.02957997 39.02957997 0 0 1 0-55.16180678 135.3025452 135.3025452 0 0 0 0-192.02553485 39.02957997 39.02957997 0 0 1 0-55.16180678 38.50918593 38.50918593 0 0 1 55.16180677 0 212.8413111 212.8413111 0 0 1 0 302.34914841 40.07036882 40.07036882 0 0 1-27.58090339 11.44867658zM1052.6962439 875.22722728a37.98879112 37.98879112 0 0 1-27.58090339-11.44867658 38.50918593 38.50918593 0 0 1 0-55.16180678 443.37603112 443.37603112 0 0 0 0-624.4732842 38.50918593 38.50918593 0 0 1 0-55.16180678 39.02957997 39.02957997 0 0 1 55.16180677 0 520.39440299 520.39440299 0 0 1 0 735.8376866 38.50918593 38.50918593 0 0 1-27.58090338 10.40788773zM680.09385154 907.4916801a97.3137533 97.3137533 0 0 1-69.73284991-29.14208628l-128.01702324-128.01702324a20.81577623 20.81577623 0 0 0-13.53025429-6.2447331H255.972413A98.87493696 98.87493696 0 0 1 156.577082 645.21290129V505.74720071a39.54997478 39.54997478 0 0 1 39.02957997-39.02957999 39.54997478 39.54997478 0 0 1 39.02958075 39.02957999v139.46570058a20.81577623 20.81577623 0 0 0 21.33617028 20.81577626H468.8137241a98.87493696 98.87493696 0 0 1 70.25324396 29.14208627l128.01702324 128.01702324a20.81577623 20.81577623 0 0 0 35.90721419-15.09143796V517.19587806a39.02957997 39.02957997 0 0 1 78.05915994 0v290.90047104a97.83414811 97.83414811 0 0 1-61.4065391 91.58941501 95.2321756 95.2321756 0 0 1-39.54997479 7.80591599z"
                      fill="#ffffff"
                      p-id="2326"></path>
                <path d="M195.60666197 565.59255769A39.54997478 39.54997478 0 0 1 156.577082 526.56297694V387.61767116A98.87493696 98.87493696 0 0 1 255.972413 288.22234015H468.8137241a20.81577623 20.81577623 0 0 0 15.09143718-6.24473232L610.36100163 153.96058459A99.395331 99.395331 0 0 1 781.05036544 224.2138293v290.90047106a39.02957997 39.02957997 0 1 1-78.05915995 0V224.2138293a21.33617028 21.33617028 0 0 0-13.00986024-19.77498738 20.81577623 20.81577623 0 0 0-22.89735395 4.68354944L537.50578516 337.1394146A98.87493696 98.87493696 0 0 1 468.8137241 366.28150088H255.972413a21.33617028 21.33617028 0 0 0-21.33617028 21.33617028v138.94530578a39.54997478 39.54997478 0 0 1-39.02958075 39.02958075z"
                      fill="#ffffff"
                      p-id="2327"></path>
              </svg>
            </span>
            <!-- 全屏控件 -->
            <button class="danmaku-button"
                    @click="fullScreen">
              <i class="el-icon-full-screen"></i>
            </button>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import './danmakuIcon.js'

export default {
  name: 'danmaku',
  props: {
    data: Array,
    src: '',
    width: {
      type: Number,
      default: 600
    },
    suit: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      width2: 0,
      height: 0,
      controls: {
        durationTime: 0,
        dragTime: -1,
        dragDelay: '',
        time1: '0:00',
        time2: '3:33',
        volume: 80,
        isDraging: false,
        show: 0,
        full: false,
        rate: 4
      },
      input: '',
      type: 'scroll',
      color: '#ffffff',
      setting: {
        onOff: true,
        block: [],
        opacity: 70,
        zone: 3,
        speed: 50,
        size: 24,
        marks: {
          0: '1/4屏',
          1: '半屏',
          2: '3/4屏',
          3: '全屏'
        }
      },
      smallScreen: false,
      tinyScreen: false,
      hideInfo: false,
      isPlaying: false,
      lastTimeStamp: 0,
      video: '',
      list: [],
      seize: [
        { top: {}, scroll: {}, bottom: {} },
        { top: {}, scroll: {}, bottom: {} },
        { top: {}, scroll: {}, bottom: {} },
        { top: {}, scroll: {}, bottom: {} },
        { top: {}, scroll: {}, bottom: {} }
      ]
    }
  },
  mounted () {
    // 初始化
    this.init()
    // 读取本地保存配置
    this.read()
    // 计时器：隐藏视频控件
    setInterval(() => {
      if (this.controls.isDraging) this.controls.show = 3
      else --this.controls.show
    }, 1000)
  },
  methods: {
    // 初始化
    init () {
      this.danmaku = document.querySelector('#danmaku')
      if (!this.danmaku) {
        setTimeout(() => this.init(), 100)
        return
      }
      this.initEventListener()
      this.setRatio()
      this.autoSuitScreen()
    },
    // 设置比例16:9
    setRatio () {
      this.width2 = this.width
      this.height = this.width2 * 9 / 16 + 30
    },
    // 绑定事件
    initEventListener () {
      this.bindVideo()
      // 视频进度条按下事件
      let slider = danmaku.querySelector('.danmaku-progress-slider .el-slider__runway')
      slider.addEventListener('mousedown', () => { this.controls.isDraging = true })
      slider.addEventListener('touchstart', () => { this.controls.isDraging = true })
    },
    // 绑定视频事件
    bindVideo () {
      this.video = danmaku.querySelector("video")
      this.video.addEventListener('playing', () => { this.isPlaying = true })
      this.video.addEventListener('pause', () => { this.isPlaying = false })
      this.video.addEventListener('timeupdate', () => this.timeupdate())
      this.video.addEventListener('seeked', () => {
        this.lastTimeStamp = Math.round(this.video.currentTime * 1000)
        this.controls.time1 = this.formatTime(this.lastTimeStamp)
        this.list = []
      })
      this.video.addEventListener('durationchange', () => {
        this.controls.durationTime = Math.round(this.video.duration * 1000)
        this.controls.time2 = this.formatTime(this.controls.durationTime)
        this.list = []
      })
    },
    // 自适应屏幕
    autoSuitScreen () {
      let main = this.danmaku.querySelector('main')
      if (this.controls.full) {
        this.width2 = main.offsetWidth
        this.height = main.offsetHeight
        setTimeout(() => {
          this.width2 = main.offsetWidth
          this.height = main.offsetHeight
        }, 100)
      } else if (this.suit) {
        if (document.body.clientWidth < this.width2) {
          this.width2 = document.body.clientWidth
          this.height = this.width2 * 9 / 16 + 60
        } else {
          this.width2 = document.body.clientWidth
          this.width2 = this.width2 > this.width ? this.width : this.width2
          this.height = this.width2 * 9 / 16 + 30
        }
      }
      this.tinyScreen = this.width2 < 315
      this.hideInfo = this.width2 < 420
      this.smallScreen = this.width2 < 680
    },
    // 监听video进度改变事件
    timeupdate () {
      const time = Math.round(this.video.currentTime * 1000)
      this.controls.time1 = this.formatTime(time)
      if (!this.controls.isDraging) { this.controls.dragTime = time }
      if (!this.setting.onOff) return
      this.data.forEach(danmu => {
        if (danmu.time >= this.lastTimeStamp && danmu.time < time) {
          this.pushDanmaku(danmu)
        }
      })
      this.lastTimeStamp = time
    },
    // 添加一条弹幕到屏幕上
    pushDanmaku (danmu) {
      danmu.id = 'danmaku' + Math.round(Math.random() * 10000000) + '_' + danmu.time
      this.list.push(danmu)
      setTimeout(() => {
        let element = document.querySelector('#' + danmu.id)
        if (!element) return
        element.style.visibility = ''
        this.calcSeize(danmu, element)
        element.addEventListener('animationend', () => {
          this.list.splice(this.list.indexOf(danmu), 1)
        })
      }, 50)
    },
    // 计算弹幕占位位置
    calcSeize (danmu, element) {
      let width = this.width2
      let height = this.height - (this.controls.full ? 0 : (this.smallScreen ? 60 : 30))
      if (danmu.type === 'scroll') height *= this.setting.zone * 0.25 + 0.25
      let position = -1
      for (let si = 0; si < 5; si++) {
        for (let i = 0; i < Math.floor(height / element.offsetHeight); i++) {
          let temp = this.seize[si][danmu.type][i]
          if (temp) {
            let el = document.querySelector('#' + temp)
            if (el) {
              if (danmu.type === 'scroll') {
                if (width - el.offsetLeft - el.offsetWidth >= 0) {
                  position = i
                } else continue
              } else continue
            } else position = i
          } else position = i
          if (position !== -1) {
            this.seize[si][danmu.type][position] = danmu.id
            if (danmu.type === 'bottom') element.style.bottom = element.offsetHeight * position + 40 + 'px'
            else element.style.top = element.offsetHeight * position + 'px'
            break
          }
        }
        if (position !== -1) break
      }
    },
    // 选择弹幕颜色
    chooseColor (color) {
      this.color = color
    },
    // 点击发送按钮
    send () {
      if (!this.video) throw Error('danmaku: 没有找到嵌入的视频标签!!')
      let danmu = {
        content: this.input,
        color: this.color,
        type: this.type,
        time: Math.round(this.video.currentTime * 1000)
      }
      this.input = ''
      this.$emit('send', danmu)
      this.data.push(danmu)
    },
    // 屏蔽弹幕
    blockType (type) {
      if (this.setting.block.indexOf(type) !== -1) {
        this.setting.block.splice(this.setting.block.indexOf(type), 1)
      } else {
        this.setting.block.push(type)
      }
    },
    // 选择要发送的弹幕类型
    sendType (type) {
      this.type = type
    },
    // 保存配置到本地
    save () {
      let save = {
        setting: this.setting,
        type: this.type,
        color: this.color,
        volume: this.controls.volume,
        rate: this.controls.rate
      }
      localStorage.setItem('danmaku', JSON.stringify(save))
    },
    // 读取本地配置
    read () {
      let read = localStorage.getItem('danmaku')
      if (!read) return
      read = JSON.parse(read)
      this.setting = read.setting
      this.type = read.type
      this.color = read.color
      this.controls.volume = read.volume
      this.controls.rate = read.rate
    },
    /* ----下面是video自定义控件的方法---- */
    // 播放、暂停
    playOrPause () {
      if (this.isPlaying) this.video.pause()
      else this.video.play()
    },
    // 格式化拖动进度条时提示的时间
    formatDragTime (value) {
      return this.formatTime(value)
    },
    // 格式化时间显示格式
    formatTime (time) {
      let str = ''
      if (time >= 3600000) str += Math.floor(time / 3600000) + ':'
      time %= 3600000
      if (time >= 60000) str += Math.floor(time / 60000) + ':'
      else str += '00:'
      time %= 60000
      let second = Math.round(time / 1000)
      if (second < 10) str += '0' + second
      else str += second
      return str
    },
    // 全屏
    fullScreen () {
      if (document.fullscreenElement || document.fullScreenElement ||
        document.mozFullScreenElement || document.mozFullscreenElement ||
        document.webkitFullScreenElement || document.webkitFullscreenElement ||
        document.msFullScreenElement || document.msFullscreenElement) {
        this.exitFullScreen()
      } else {
        let d = this.danmaku.querySelector('main')
        var requestMethod = d.requestFullScreen || d.webkitRequestFullScreen ||
          d.mozRequestFullScreen || d.msRequestFullScreen
        requestMethod.call(d)
        this.controls.full = true
        this.setRatio()
        this.autoSuitScreen()
      }
    },
    // 退出全屏
    exitFullScreen () {
      this.controls.full = false
      document.exitFullscreen()
      this.setRatio()
      this.autoSuitScreen()
    },
    // 拖动进度条后，松开鼠标时触发
    dragEng () {
      clearInterval(this.controls.dragDelay)
      this.controls.dragDelay = setTimeout(() => {
        this.controls.isDraging = false
      }, 1000)
    }
  },
  computed: {
    aniState () {
      return this.isPlaying ? 'running' : 'paused'
    },
    formatRate () {
      if (this.controls.rate === 4) return '倍速'
      return this.controls.rate / 4 + 'x'
    },
    danmakuStyle () {
      return 'width:' + this.width2 + 'px;height:' + this.height + 'px'
    }
  },
  watch: {
    setting: {
      deep: true,
      handler () { this.save() }
    },
    'setting.onOff' (value) {
      if (!value) this.list = []
    },
    'controls.dragTime' (value) {
      if (this.controls.isDraging) {
        this.video.currentTime = value / 1000
      }
    },
    'controls.volume' (value) {
      this.video.volume = value / 100
      this.save()
    },
    'controls.rate' (value) {
      this.video.playbackRate = value / 4
      this.save()
    },
    type () { this.save() },
    color () { this.save() }
  }
}
</script>

<style lang="scss" scoped>
@mixin danmaku-screen {
  width: 100%;
  height: calc(100% - 30px);
  #danmaku.smallScreen & {
    height: calc(100% - 60px);
  }
  #danmaku.full & {
    height: 100%;
  }
}
#danmaku {
  position: relative;
  display: inline-block;
  background-color: black;
  color: white;
}
.danmaku-video {
  position: absolute;
  @include danmaku-screen;
  top: 0;
  left: 0;
}
.danmaku-main {
  position: relative !important;
  height: 100%;
  width: 100%;
  background-color: transparent;
  top: 0;
  left: 0;
}

.danmaku-danmu-container {
  @include danmaku-screen;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 2;
  font-weight: bold;
  overflow: hidden;
  &.top .danmaku-danmu.top,
  &.bottom .danmaku-danmu.bottom,
  &.scroll .danmaku-danmu.scroll {
    visibility: hidden;
  }
  & .danmaku-danmu {
    white-space: nowrap;
    position: absolute;
    display: inline-block;
    user-select: none;
    cursor: default;
    &:hover {
      text-shadow: 0 0 1px white, 0 0 3px white, 0 0 6px white, 0 0 10px white;
    }
    &.top,
    &.bottom {
      left: 50%;
      transform: translateX(-50%);
    }
  }
}
.danmaku-controls-container {
  position: absolute;
  width: 100%;
  height: 30px;
  z-index: 3;
  bottom: 0;
  background: black;
  #danmaku.smallScreen & {
    height: 60px;
  }
  #danmaku.full & {
    position: absolute;
    bottom: 0;
    left: 0;
    background: rgba(black, 0.5);
  }
  & .dammaku-controls {
    position: relative;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    text-align: center;
    white-space: nowrap;
    & .danmaku-button {
      pointer-events: all;
      height: 30px;
      color: white;
      background: transparent;
      border: none;
      outline: none;
      cursor: pointer;
      font-size: 24px;
      &:hover {
        color: #f383ae;
      }
    }
    & .danmaku-float-left {
      float: left;
      & .danmaku-time {
        pointer-events: none;
        display: inline-block;
        position: relative;
        top: -2px;
      }
    }
    & .danmaku-float-right {
      float: right;
      & .danmaku-rate {
        & .danmaku-tooltip-content {
          width: 5px;
          height: 140px;
          padding: 5px 13px;
          /deep/ & .el-slider__runway {
            margin: 0;
            margin-top: 3px;
            & .el-slider__bar {
              background: rgb(223, 161, 189);
            }
          }
          /deep/ & .el-slider__button {
            width: 7px;
            height: 7px;
            border: 2px solid rgb(255, 47, 47);
            &.hover {
              width: 10px;
              height: 10px;
            }
            &:active {
              width: 10px;
              height: 10px;
            }
          }
          &::after {
            border: 7px solid;
            border-color: rgba(87, 87, 87, 0.87) transparent transparent;
            bottom: -13px;
            transform: translateX(-8px);
          }
        }
      }
      & .danmaku-volume {
        width: 40px;
        & .danmaku-tooltip-content {
          width: 5px;
          height: 90px;
          padding: 5px 13px;
          /deep/ & .el-slider__runway {
            margin: 0;
            margin-top: 3px;
            & .el-slider__bar {
              background: rgb(224, 132, 90);
            }
          }
          /deep/ & .el-slider__button {
            width: 7px;
            height: 7px;
            border: 2px solid rgb(255, 165, 47);
            &.hover {
              width: 10px;
              height: 10px;
            }
            &:active {
              width: 10px;
              height: 10px;
            }
          }
          &::after {
            border: 7px solid;
            border-color: rgba(87, 87, 87, 0.87) transparent transparent;
            bottom: -13px;
            transform: translateX(-8px);
          }
        }
      }
      & .danmaku-button {
        vertical-align: top;
      }
    }
    & .danmaku-info {
      user-select: none;
      font-size: 14px;
      vertical-align: middle;
    }
    & .danmaku-switch {
      vertical-align: top;
      height: 30px;
      /deep/ & .el-switch__core::after {
        content: "弹";
        color: white;
        font-size: 12px;
        line-height: 16px;
        background: orchid;
      }
    }
    & .danmaku-tooltip {
      position: relative;
      width: 30px;
      height: 30px;
      display: inline-block;
      line-height: 30px;
      vertical-align: middle;
      fill: currentColor;
      & i {
        font-size: 20px;
      }
      & .danmaku-tooltip-span {
        cursor: default;
      }
      & .danmaku-tooltip-content {
        display: none;
        background-color: rgba(87, 87, 87, 0.87);
        border-radius: 3px;
        font-size: 12px;
        padding: 5px;
        position: absolute;
        top: -10px;
        left: 15px;
        z-index: 99;
        transform: translate(-50%, -100%);
        &::after {
          content: " ";
          border: 10px solid;
          border-color: rgba(87, 87, 87, 0.87) transparent transparent
            transparent;
          position: absolute;
          bottom: -19px;
          transform: translateX(-10px);
        }
      }
      &:hover .danmaku-tooltip-content {
        display: block;
      }
    }
    & .danmaku-setting {
      & i {
        vertical-align: middle;
      }
      & .danmaku-tooltip-content {
        width: 250px;
        & .danmaku-block {
          & .danmaku-block-button {
            position: relative;
            top: 0;
            padding: 0;
            width: 40px;
            height: 55px;
            padding: 0;
            background: transparent;
            color: #bcbcbc;
            outline: none;
            border: none;
            margin: 0 6px;
            cursor: pointer;
            &:hover,
            &:active,
            &.active {
              color: #00afff;
            }
            & .icon {
              width: 40px;
              height: 30px;
              fill: currentColor;
              overflow: hidden;
            }
            & .danmaku-block-button-explain {
              width: 40px;
              height: 20px;
            }
          }
        }
        & .danmaku-line {
          margin: 10px 0;
          white-space: nowrap;
          & .danmaku-line-label {
            width: 60px;
            vertical-align: middle;
          }
          & .danmaku-line-slider {
            vertical-align: top;
            display: inline-block;
            width: calc(100% - 70px);
            margin: 0 5px;
            /deep/ & .el-slider__runway {
              margin: 0;
            }
          }
        }
        & .danmaku-line2 {
          height: 40px;
        }
      }
    }
    & .danmaku-input {
      width: calc(100% - 480px);
      max-width: 500px;
      font-size: 14px;
      height: 20px;
      vertical-align: top;
      margin-top: 2px;
      border-radius: 4px;
      background: linear-gradient(
        rgba(rgb(80, 110, 192), 0.5),
        rgba(rgb(22, 151, 97), 0.5)
      );
      /deep/ & .el-input-group__prepend {
        background: transparent;
        padding: 0;
      }
      /deep/ & .el-input-group__append {
        background: transparent;
        width: 50px;
        text-align: center;
        padding: 0;
        color: white;
        &:hover {
          color: rgb(255, 143, 180);
        }
      }
      /deep/ & .el-input__inner {
        height: 26px;
        color: white;
        background: transparent;
      }
      #danmaku.smallScreen & {
        width: calc(100% - 10px);
        left: 0;
        margin-left: 5px;
        bottom: 2px;
        padding: 0;
        position: absolute;
        max-width: 99999px;
      }
      #danmaku.tinyScreen & {
        width: calc(100% - 125px);
      }
      #danmaku.full & {
        background: rgba(black, 0.5);
      }
      & .danmaku-input-setting {
        height: 20px;
        & i {
          vertical-align: top;
        }
        & .danmaku-tooltip-content {
          width: 160px;
          #danmaku.smallScreen & {
            transform: translate(-20px, -100%);
            &::after {
              transform: translateX(-75px);
            }
          }
          & .danmaku-input-setting-block {
            & .danmaku-block-button {
              position: relative;
              top: 0;
              padding: 0;
              width: 40px;
              height: 55px;
              padding: 0;
              background: transparent;
              color: #bcbcbc;
              outline: none;
              border: none;
              margin: 0 6px;
              cursor: pointer;
              &:hover,
              &:active,
              &.active {
                color: #00afff;
              }
              & .icon {
                width: 40px;
                height: 30px;
                fill: currentColor;
                overflow: hidden;
              }
              & .danmaku-block-explain {
                width: 40px;
                height: 20px;
              }
            }
          }
          & .danmaku-input-setting-line {
            & .danmaku-input-setting-label {
              text-align: center;
              display: inline-block;
              vertical-align: top;
            }
            & .danmaku-input-setting-color {
              width: 15px;
              height: 15px;
              border-radius: 3px;
              background: white;
              outline: none;
              border: none;
              cursor: pointer;
              &.blue {
                background: #00afff;
              }
              &.red {
                background: #ff0000;
              }
              &.yellow {
                background: #ffd700;
              }
              &.green {
                background: #008000;
              }
              &.active {
                box-shadow: 0 0 2px white, 0 0 5px white;
              }
            }
          }
        }
      }
      & .danmaku-input-send {
        width: 100%;
        height: 100%;
        line-height: 100%;
        font-size: 15px;
        margin: 0;
        padding: 0;
      }
    }
  }
}
.danmaku-progress-container {
  position: absolute;
  width: 100%;
  top: -90px;
  left: 0;
  height: 90px;
  overflow: hidden;
  pointer-events: none;
  & .danmaku-progress {
    position: relative;
    left: 0;
    top: 0;
    height: 100%;
    width: 100%;
    & .danmaku-progress-slider {
      z-index: 2;
      opacity: 0.8;
      position: absolute;
      left: 25px;
      bottom: 0;
      height: 30px;
      width: calc(100% - 50px);
      margin: 0 auto;
      pointer-events: all;
      /deep/ & .el-slider__runway {
        margin: 0;
        margin-top: 12px;
      }
      /deep/ & .el-slider__button {
        width: 7px;
        height: 7px;
        &:hover,
        &:active {
          width: 10px;
          height: 10px;
        }
      }
      &.danmaku-progress-hidden {
        width: 100%;
        left: 0;
        bottom: -17px;
        /deep/ & .el-slider__button {
          width: 0;
          height: 0;
        }
      }
    }
  }
}
</style>

<style lang="scss">
@keyframes danmakuscroll {
  0% {
    left: 100%;
    transform: translateX(10%);
  }
  100% {
    left: 0;
    transform: translateX(-100%);
  }
}
@keyframes danmakutop {
  0% {
    border-radius: 0;
  }
  100% {
    border-radius: 0;
  }
}
@keyframes danmakubottom {
  0% {
    border-radius: 0;
  }
  100% {
    border-radius: 0;
  }
}
</style>
