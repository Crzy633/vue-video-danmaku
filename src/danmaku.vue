<template>
  <div id="danmaku"
       :class="[{ full: controls.full }, { smallScreen }]"
       :style="danmakuStyle">
    <div class="slot-container">
      <video class="video"
             :src="src"></video>
      <div @click="playOrPause"
           class="preventClick"></div>
      <div class="controls">
        <div class="hoverLine"
             @click="playOrPause"
             @mousemove="controls.show = 3"></div>
        <div class="timeLine"
             @mousedown="controls.isDraging = true"
             :style="'bottom:' + (controls.show > 0 ? '0' : (controls.full ? '-99px' : '-22px'))">
          <el-slider v-model="controls.dragTime"
                     class="videoSlider"
                     :max="controls.durationTime"
                     @change="dragEng"
                     :format-tooltip="formatDragTime"></el-slider>
        </div>
      </div>
    </div>
    <div :class="['canvas'].concat(setting.block)"
         id="danmakuCanvas"
         v-if="setting.onOff">
      <div v-for="danmu in list"
           :id="danmu.id"
           :key="danmu.id"
           :class="['danmu', danmu.type]"
           :style="'color:' + danmu.color +
                   ';opacity:' + setting.opacity/100 +
                   ';font-size:' + setting.size + 'px' +
                   ';animation: danmaku' + danmu.type + ' linear ' + (((100 - setting.speed) / 5) - (controls.rate - 4) * 0.75) + 's 1 forwards' +
                   ';animation-play-state: ' + aniState">{{danmu.content}}</div>
    </div>
    <div :class="['main', { smallScreen }]"
         @mousemove="controls.show = 3"
         :style="'bottom:' + (controls.full ? (controls.show > 0 ? '0' : '-99px') : '0')">
      <div class="float-left">
        <button class="button"
                @click="playOrPause">
          <i v-if="!isPlaying"
             class="el-icon-video-play"></i>
          <i v-if="isPlaying"
             class="el-icon-video-pause"></i>
        </button>
        <span class="time">
          <span>{{controls.time1}}/{{controls.time2}}</span>
        </span>
      </div>
      <el-switch class="switch"
                 v-model="setting.onOff"></el-switch>
      <span class="tooltip-icon">
        <i class="el-icon-c-scale-to-original"></i>
        <div class="danmaku-tooltip-content">
          <div>按类型屏蔽</div>
          <div class="block">
            <button :class="['block-button', { active: setting.block.indexOf('scroll') !== -1 }]"
                    @click="blockType('scroll')">
              <svg class="icon"
                   aria-hidden="true">
                <use xlink:href="#icon-bofangqi-danmugundongkai"></use>
              </svg>
              <br>
              <span class="block-explain">滚动</span>
            </button>
            <button :class="['block-button', { active: setting.block.indexOf('top') !== -1 }]"
                    @click="blockType('top')">
              <svg class="icon"
                   aria-hidden="true">
                <use xlink:href="#icon-bofangqi-danmudingbukai"></use>
              </svg>
              <br>
              <span class="block-explain">顶部</span>
            </button>
            <button :class="['block-button', { active: setting.block.indexOf('bottom') !== -1 }]"
                    @click="blockType('bottom')">
              <svg class="icon"
                   aria-hidden="true">
                <use xlink:href="#icon-bofangqi-danmudibukai"></use>
              </svg>
              <br>
              <span class="block-explain">底部</span>
            </button>
          </div>
          <div class="line">
            <span class="label">不透明度</span>
            <el-slider class="slider"
                       :show-tooltip="false"
                       v-model="setting.opacity"></el-slider>
          </div>
          <div class="line line2">
            <span class="label">显示区域</span>
            <el-slider class="slider"
                       :max="3"
                       :show-stops="true"
                       :show-tooltip="false"
                       :marks="setting.marks"
                       v-model="setting.zone"></el-slider>
          </div>
          <div class="line">
            <span class="label">弹幕速度</span>
            <el-slider class="slider"
                       :min="1"
                       :max="100"
                       :show-tooltip="false"
                       v-model="setting.speed"></el-slider>
          </div>
          <div class="line">
            <span class="label">字体大小</span>
            <el-slider class="slider"
                       :min="12"
                       :max="48"
                       :show-tooltip="false"
                       v-model="setting.size"></el-slider>
          </div>
        </div>
      </span>
      <el-input v-model="input"
                class="input"
                :maxlength="50"
                placeholder="发个弹幕~"
                @keyup.enter.native="send">
        <span class="tooltip-icon tooltip-icon2"
              slot="prepend">
          <i class="el-icon-s-open"></i>
          <div class="danmaku-tooltip-content">
            <div>类型</div>
            <div class="block">
              <button :class="['block-button', { active: type==='scroll' }]"
                      @click="sendType('scroll')">
                <svg class="icon"
                     aria-hidden="true">
                  <use xlink:href="#icon-bofangqi-danmugundongkai"></use>
                </svg>
                <br>
                <span class="block-explain">滚动</span>
              </button>
              <button :class="['block-button', { active: type==='top' }]"
                      @click="sendType('top')">
                <svg class="icon"
                     aria-hidden="true">
                  <use xlink:href="#icon-bofangqi-danmudingbukai"></use>
                </svg>
                <br>
                <span class="block-explain">顶部</span>
              </button>
              <button :class="['block-button', { active: type==='bottom' }]"
                      @click="sendType('bottom')">
                <svg class="icon"
                     aria-hidden="true">
                  <use xlink:href="#icon-bofangqi-danmudibukai"></use>
                </svg>
                <br>
                <span class="block-explain">底部</span>
              </button>
            </div>
            <div class="line">
              <span class="label">颜色</span>
              <button @click="chooseColor('#ffffff')"
                      :class="['color', {active: color==='#ffffff'}]"></button>
              <button @click="chooseColor('#00afff')"
                      :class="['color', 'blue', {active: color==='#00afff'}]"></button>
              <button @click="chooseColor('#ff0000')"
                      :class="['color', 'red', {active: color==='#ff0000'}]"></button>
              <button @click="chooseColor('#ffd700')"
                      :class="['color', 'yellow', {active: color==='#ffd700'}]"></button>
              <button @click="chooseColor('#008000')"
                      :class="['color', 'green', {active: color==='#008000'}]"></button>
            </div>
          </div>
        </span>
        <el-button class="send"
                   @click="send"
                   slot="append">发送</el-button>
      </el-input>
      <span class="info"
            v-if="!hideInfo">
        <span v-if="data.length > 0">{{data.length}}条弹幕</span>
      </span>
      <div class="float-right">
        <span class="tooltip-icon tooltip-icon3 tooltip-icon4">
          <div class="danmaku-tooltip-content">
            <el-slider v-model="controls.rate"
                       class="volumeSlider"
                       :min="1"
                       :max="12"
                       vertical
                       :show-tooltip="false"
                       height="130px"></el-slider>
          </div>
          <span class="formatRate">{{formatRate}}</span>
        </span>
        <span class="tooltip-icon tooltip-icon3">
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
        <button class="button"
                @click="fullScreen">
          <i class="el-icon-full-screen"></i>
        </button>
      </div>
    </div>
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
    init () {
      let danmaku = document.querySelector('#danmaku')
      if (!danmaku) setTimeout(() => this.init(), 100)
      // 获取视频标签<video>
      let video = danmaku.querySelector('video')
      if (video === null) throw Error('danmaku: 没有找到嵌入的视频标签!!')
      else this.video = video
      // 绑定视频事件
      video.addEventListener('playing', () => { this.isPlaying = true })
      video.addEventListener('pause', () => { this.isPlaying = false })
      video.addEventListener('timeupdate', () => this.timeupdate())
      video.addEventListener('seeked', () => {
        this.lastTimeStamp = Math.round(this.video.currentTime * 1000)
        this.controls.time1 = this.formatTime(this.lastTimeStamp)
        this.list = []
      })
      video.addEventListener('durationchange', () => {
        this.controls.durationTime = Math.round(this.video.duration * 1000)
        this.controls.time2 = this.formatTime(this.controls.durationTime)
        this.list = []
      })
      // 监听全屏
      let resize = window.onresize
      window.onresize = () => {
        if (resize) resize()
        if (document.fullscreenElement) this.controls.full = true
        else this.controls.full = false
        this.autoSuitScreen()
      }
      // 设置宽高，16：9
      this.width2 = this.width
      this.height = this.width2 * 9 / 16 + 30
      this.autoSuitScreen()
    },
    // 自适应屏幕
    autoSuitScreen () {
      if (this.suit) {
        if (document.body.clientWidth < this.width2) {
          this.width2 = document.body.clientWidth
          this.height = this.width2 * 9 / 16 + 60
        } else {
          this.width2 = document.body.clientWidth
          this.width2 = this.width2 > this.width ? this.width : this.width2
          this.height = this.width2 * 9 / 16 + 30
        }
      }
      this.hideInfo = this.width2 < 380
      this.smallScreen = this.width2 < 600
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
        element.style.visibility = 'visible'
        this.calcSeize(danmu, element)
        element.addEventListener('animationend', () => {
          this.list.splice(this.list.indexOf(danmu), 1)
        })
      }, 50)
    },
    // 计算屏幕占位位置
    calcSeize (danmu, element) {
      let canvas = document.querySelector('#danmakuCanvas')
      let width = canvas.offsetWidth
      let height = canvas.offsetHeight
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
      localStorage.setItem("danmaku", JSON.stringify(save))
    },
    // 读取本地配置
    read () {
      let read = localStorage.getItem("danmaku")
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
      if (document.fullscreenElement) document.exitFullscreen()
      else {
        let d = document.querySelector('#danmaku')
        var requestMethod = d.requestFullScreen || d.webkitRequestFullScreen ||
          d.mozRequestFullScreen || d.msRequestFullScreen
        requestMethod.call(d)
      }
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
#danmaku {
  position: relative;
  display: inline-block;
  background-color: black;
  color: white;
}
.video {
  position: absolute;
  width: 100%;
  height: 100%;
  left: 0;
}
.preventClick {
  position: relative;
  top: 0;
  width: 100%;
  height: 100%;
  z-index: 2;
}
.controls {
  pointer-events: none;
  z-index: 3;
  position: absolute;
  width: 100%;
  height: 60px;
  bottom: -25px;
  & .hoverLine {
    pointer-events: all;
    z-index: -1;
    position: absolute;
    width: 100%;
    height: 100%;
    top: 0;
  }
  & .timeLine {
    opacity: 0.8;
    pointer-events: all;
    position: relative;
    padding: 0 20px;
    height: 30px;
    & .videoSlider {
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
    }
  }
}
.slot-container {
  position: relative;
  width: 100%;
  height: calc(100% - 30px);
  left: 0;
  top: 0;
  background: black;
  border: none;
  outline: none;
  overflow: hidden;
}
.canvas {
  position: absolute;
  width: 100%;
  height: calc(100% - 30px);
  left: 0;
  top: 0;
  pointer-events: none;
  overflow: hidden;
  font-weight: bold;
  &.top .danmu.top {
    visibility: hidden;
  }
  &.bottom .danmu.bottom {
    visibility: hidden;
  }
  &.scroll .danmu.scroll {
    visibility: hidden;
  }
}
.danmu {
  visibility: hidden;
  white-space: nowrap;
  position: absolute;
  display: inline-block;
  pointer-events: all;
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
.main {
  z-index: 99;
  width: 100%;
  position: relative;
  height: 30px;
  line-height: 30px;
  text-align: center;
  white-space: nowrap;
  background: black;
  & .button {
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
  & .float-left {
    float: left;
    & .time {
      pointer-events: none;
      vertical-align: top;
    }
  }
  & .float-right {
    float: right;
    & .tooltip-icon {
      pointer-events: all;
      display: inline-block;
      position: relative;
      top: 0px;
      & .icon {
        margin-top: 0px;
        color: white;
        width: 30px;
        height: 30px;
        &:hover {
          color: red;
        }
      }
    }
    & .button {
      vertical-align: top;
    }
  }
  & .info {
    user-select: none;
    font-size: 14px;
    vertical-align: top;
  }
  & .switch {
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
  & .tooltip-icon {
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
    & .danmaku-tooltip-content {
      display: none;
      background-color: rgba(87, 87, 87, 0.87);
      border-radius: 3px;
      font-size: 12px;
      width: 250px;
      padding: 5px;
      position: absolute;
      top: -10px;
      left: 15px;
      transform: translate(-50%, -100%);
      &::after {
        content: " ";
        border: 10px solid;
        border-color: rgba(87, 87, 87, 0.87) transparent transparent transparent;
        position: absolute;
        bottom: -20px;
        transform: translateX(-10px);
      }
      & .block {
        & .block-button {
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
          & .block-explain {
            width: 40px;
            height: 20px;
          }
        }
      }
      & .line {
        margin: 10px 0;
        white-space: nowrap;
        & .label {
          width: 60px;
          vertical-align: middle;
        }
        & .slider {
          vertical-align: top;
          display: inline-block;
          width: calc(100% - 70px);
          margin: 0 5px;
          & .el-slider__runway {
            margin: 10px 0;
          }
        }
      }
      & .line2 {
        height: 40px;
      }
    }
    &:hover .danmaku-tooltip-content {
      display: block;
    }
  }
  & .tooltip-icon2 {
    height: 20px;
    & .danmaku-tooltip-content {
      width: 160px;
      & .line {
        & .label {
          text-align: center;
          display: inline-block;
          vertical-align: top;
        }
        & .color {
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
  & .tooltip-icon3 {
    height: 30px;
    & .danmaku-tooltip-content {
      display: absolute;
      width: 5px;
      height: 90px;
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
    }
  }
  & .tooltip-icon4 {
    width: 40px;
    height: 30px;
    & .danmaku-tooltip-content {
      display: absolute;
      width: 5px;
      height: 140px;
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
    }
    & .formatRate {
      cursor: default;
    }
  }
  & .input {
    width: calc(100% - 380px);
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
      padding: 0 10px;
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
    & .tooltip-send {
      width: 20px;
      height: 20px;
      font-size: 16px;
    }
  }
}
.smallScreen {
  & .controls {
    height: 90px;
    bottom: -55px;
  }
  & .slot-container {
    height: calc(100% - 60px);
  }
  & .main {
    height: 60px;
    & .input {
      width: calc(100% - 10px);
      left: 0;
      bottom: 2px;
      padding: 0 5px;
      position: absolute;
      max-width: 999999px;
      & .danmaku-tooltip-content {
        transform: translate(-20px, -100%);
        &::after {
          transform: translateX(-75px);
        }
      }
    }
  }
}
.full {
  & .controls {
    bottom: 5px;
  }
  & .slot-container {
    height: 100%;
  }
  & .main {
    background: rgba($color: #000000, $alpha: 0.5);
    position: absolute;
    bottom: 0;
    & .input {
      background: rgba(black, 0.5);
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
