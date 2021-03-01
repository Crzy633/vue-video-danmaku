<template>
  <div class="photo">
    <slot></slot>
    <label for="photoUploadInput"></label>
    <input id="photoUploadInput"
           type="file"
           class="upload"
           @change="fileChange"
           :accept="accepts">
    <div class="crop"
         v-if="showCrop"
         transition="crop">
      <div class="dialog"
           :style="{width:rWidth+'px'}"
           v-if="showCropDialog">
        <section :style="{width:rWidth+'px',height:rHeight+'px'}"
                 :class="{'grab':grabbing==false,'grabbing':grabbing}">
          <img :style="{width:scaleWidth+'px',height:scaleHeight+'px',left:scaleX+'px',top:scaleY+'px'}"
               @mousedown.prevent="cropDragBegin"
               @mousemove.prevent="cropDragMove"
               @mouseup.prevent="cropDragEnd"
               @touchstart.prevent="cropDragBegin"
               @touchmove.prevent="cropDragMove"
               @touchend.prevent="cropDragEnd"
               :src="cropImage">
          <div class="cropBorder"></div>
        </section>
        <footer>
          <button @click="cropCancel">取消</button>
          <button @click="cropOK">裁剪</button>
        </footer>
        <input type="range"
               v-model="scale"
               min="100"
               max="200"
               step="1"
               :class="{center:!showRatio}"
               @input="cropZoom">
        <div class="ratio"
             v-if="showRatio">
          <button @click="ratioChange">4:3</button>
          <button @click="ratioChange">1:1</button>
          <button @click="ratioChange">16:9</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.photo {
  position: relative;
  .image {
    transition: all 0.3s ease-in-out;
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    background-size: cover;
    background-position: center;
    background-repeat: no-repeat;
    border-radius: inherit;
  }
  & > input {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    opacity: 0;
    height: 0;
  }
  & > label {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    opacity: 0;
    height: 100%;
    width: 100%;
  }
  .crop {
    position: fixed;
    z-index: 9998;
    width: 100vw;
    height: 100vh;
    left: 0;
    top: 0;
    background-color: rgba(0, 0, 0, 0.7);
    transition: opacity 0.5s ease;
    backdrop-filter: blur(5px);
    overflow: hidden;
    .dialog {
      position: fixed;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: #fafafc;
      border-radius: 3px;
      transition: all 0.5s ease;

      .ratio {
        height: 24px;
        white-space: nowrap;
        line-height: 24px;
        position: absolute;
        top: -30px;
        left: 50%;
        z-index: 9999;
        font-size: 18px;
        button {
          margin: 5px;
        }
      }

      section {
        display: block;
        position: relative;
        -webkit-user-select: none;
        user-select: none;
        border-radius: 3px 3px 0 0;

        img {
          position: absolute;
        }

        .cropBorder {
          position: absolute;
          z-index: 9998;
          width: calc(100% - 3px);
          height: calc(100% - 3px);
          top: 50%;
          left: 50%;
          transform: translate(-50%, -50%);
          border-top: 50vh solid rgba(0, 0, 0, 0.8);
          border-bottom: 50vh solid rgba(0, 0, 0, 0.8);
          border-left: 50vw solid rgba(0, 0, 0, 0.8);
          border-right: 50vw solid rgba(0, 0, 0, 0.8);
          pointer-events: none;
        }
      }
      .grab {
        cursor: grab;
      }
      .grabbing {
        cursor: grabbing;
      }

      input {
        position: absolute;
        z-index: 9999;
        top: -15px;
        left: calc(50% - 150px);
        &.center {
          left: 50%;
          transform: translate(-50%);
        }
      }

      footer {
        position: relative;
        z-index: 9999;
        display: block;
        border-top: 1px solid #e8e8ea;

        button {
          float: left;
          width: 50%;
          height: 100%;
          padding: 1rem 0;
          font-weight: 600;
          text-align: center;
          color: #5f7b88;
          cursor: pointer;
          font-size: 14px;
          background-color: white;
          border: 0;
          user-select: none;
          &:nth-child(1) {
            border-right: 1px solid #e8e8ea;
          }
        }
      }
    }
    .loading {
      position: absolute;
      left: 50%;
      top: 50%;
      transform: translate(-50%, -50%);
      .v-beat {
        animation: v-beatStretchDelay 0.7s infinite linear;
        animation-fill-mode: both;
        display: inline-block;
        width: 10px;
        height: 10px;
        background-color: #fff;
        border-radius: 50%;
      }
      .v-beat-odd {
        animation-delay: 0s;
      }
      .v-beat-even {
        animation-delay: 0.35s;
      }
    }
  }
}

@-webkit-keyframes v-beatStretchDelay {
  50% {
    -webkit-transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    -webkit-transform: scale(1);
    opacity: 1;
  }
}

@-moz-keyframes v-beatStretchDelay {
  50% {
    -webkit-transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    -webkit-transform: scale(1);
    opacity: 1;
  }
}

@-ms-keyframes v-beatStretchDelay {
  50% {
    -webkit-transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    -webkit-transform: scale(1);
    opacity: 1;
  }
}

@-o-keyframes v-beatStretchDelay {
  50% {
    -webkit-transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    -webkit-transform: scale(1);
    opacity: 1;
  }
}

@keyframes v-beatStretchDelay {
  50% {
    -webkit-transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    -webkit-transform: scale(1);
    opacity: 1;
  }
}

@-webkit-keyframes v-beatStretchDelay {
  50% {
    transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@-moz-keyframes v-beatStretchDelay {
  50% {
    transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@-ms-keyframes v-beatStretchDelay {
  50% {
    transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@-o-keyframes v-beatStretchDelay {
  50% {
    transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes v-beatStretchDelay {
  50% {
    transform: scale(0.75);
    opacity: 0.2;
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

input[type="range"] {
  -webkit-appearance: none;
  &:focus {
    outline: none;
    &::-ms-fill-lower {
      background: rgba(0, 0, 0, 0.5);
    }
    &::-ms-fill-upper {
      background: rgba(0, 0, 0, 0.5);
    }
  }
  &::-webkit-slider-runnable-track {
    width: 100%;
    height: 2px;
    cursor: pointer;
    background: #efb708;
  }
  &::-webkit-slider-thumb {
    height: 20px;
    width: 20px;
    border-radius: 10px;
    background: #ffffff;
    border: 2px solid #efb708;
    cursor: pointer;
    -webkit-appearance: none;
    margin-top: -10px;
  }
  &::-moz-range-track {
    width: 100%;
    height: 2px;
    cursor: pointer;
    background: #efb708;
  }
  &::-moz-range-thumb {
    height: 16px;
    width: 16px;
    border-radius: 8px;
    border: 2px solid #efb708;
    background: #ffffff;
    cursor: pointer;
  }
  &::-ms-track {
    width: 100%;
    height: 1px;
    cursor: pointer;
    background: transparent;
    border-color: transparent;
    color: transparent;
  }
  &::-ms-fill-lower {
    background: rgba(0, 0, 0, 0.5);
    border: 0px solid rgba(200, 200, 200, 0.2);
    border-radius: 0px;
    box-shadow: 0px 0px 0px rgba(0, 0, 0, 0), 0px 0px 0px rgba(13, 13, 13, 0);
  }
  &::-ms-fill-upper {
    background: rgba(0, 0, 0, 0.5);
    border: 0px solid rgba(200, 200, 200, 0.2);
    border-radius: 0px;
    box-shadow: 0px 0px 0px rgba(0, 0, 0, 0), 0px 0px 0px rgba(13, 13, 13, 0);
  }
  &::-ms-thumb {
    height: 16px;
    width: 16px;
    border-radius: 8px;
    background: #ffffff;
    cursor: pointer;
    height: 1px;
  }
}
</style>

<script>
export default {
  props: {
    // Upload input filename,used for server side get the file stream.
    filename: {
      type: String,
      default: 'file'
    },
    // Allow Upload extension
    ext: {
      type: String,
      default: 'jpg,gif,png'
    },
    // File upload limit
    limit: {
      type: Number,
      default: 1024 * 1024 * 64
    },
    // Crop image quality
    quality: {
      type: Number,
      default: 1
    },
    // 比例，例:"16:9"
    ratio: {
      type: String,
      default: ''
    },
    // 限制最大宽度，再配合上面的"比例"，即可限制最大高度
    maxWidth: {
      type: Number,
      default: 350
    },
  },
  data () {
    return {
      width: 350,
      height: 350,
      showRatio: true,
      files: [],
      accepts: '',
      mimes: {
        'jpg': 'image/jpeg',
        'png': 'image/png',
        'gif': 'image/gif',
        'mp4': 'video/mp4',
        'mov': 'video/quicktime',
        'wmv': 'video/x-ms-wmv',
        'flv': 'video/x-flv',
        'svg': 'image/svg+xml',
        'psd': 'image/photoshop',
        'mp3': 'audio/mpeg',
        'rar': 'application/x-rar-compressed',
        'zip': 'application/zip',
        'json': 'application/json',
        'docx': 'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
        'xlsx': 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        'pptx': 'application/vnd.openxmlformats-officedocument.presentationml.presentation',
        'doc': 'application/msword',
        'pdf': 'application/pdf',
        'xls': 'application/vnd.ms-excel',
        'ppt': 'application/vnd.ms-powerpoint'
      },
      input: null,
      showCrop: false,
      showCropDialog: false,
      // 缩放
      scale: 100,
      scaleX: 0,
      scaleY: 0,
      scaleLength: 0,
      scaleWidth: 0,
      scaleHeight: 0,
      drag: false,
      grabbing: false,
      beginX: 0,
      beginY: 0,
      beginLength: 0,
      img: null,
      canvas: null,
      context: null
    }
  },
  computed: {
    rWidth () {
      return this.width
    },
    rHeight () {
      return this.height
    },
    isPE () {
      var userAgentInfo = navigator.userAgent
      var Agents = ['Android', 'iPhone', 'SymbianOS', 'Windows Phone', 'iPad', 'iPod']
      var flag = false
      for (var v = 0; v < Agents.length; v++) {
        if (userAgentInfo.indexOf(Agents[v]) > 0) {
          flag = true
          break
        }
      }
      return flag
    }
  },
  mounted () {
    this.width = this.maxWidth
    let accepts = []
    let exts = this.ext.split(',')
    for (let i = 0; i < exts.length; i++) {
      accepts.push(this.mimes[exts[i]])
    }
    this.accepts = accepts.join(',')
    // 若有设置固定比例 则隐藏切换比例按钮
    this.showRatio = this.ratio ? false : true
    // 修改比例
    this.setRatio()
    // 创建画布
    if (!this.canvas) {
      this.canvas = document.createElement('canvas')
      this.context = this.canvas.getContext('2d')
    }
  },
  methods: {
    ratioChange (val) {
      // 获取按钮中的值
      this.setRatio(val.target.innerHTML)
      this.scaleComponent.call(this)
    },
    setRatio (ratio) {
      ratio = ratio ? ratio : this.ratio ? this.ratio : "1:1"
      ratio = ratio.split(':').map(i => parseInt(i))
      let bodyWidth = document.documentElement.clientWidth
      let bodyHeight = document.documentElement.clientHeight
      let r = ratio[0] / ratio[1]
      this.height = this.width / r
      if (ratio[0] >= ratio[1]) {
        if (this.width > bodyWidth) {
          this.width = bodyWidth
        }
        this.height = this.width / r
        if (this.height > bodyHeight - 100) {
          this.height = bodyHeight - 100
        }
        this.width = this.height * r
      } else {
        if (this.height > bodyHeight - 100) {
          this.height = bodyHeight - 100
        }
        this.width = this.height * r
        if (this.width > bodyWidth) {
          this.width = bodyWidth
        }
        this.height = this.width / r
      }
    },
    scaleComponent () {
      let min = this.getMinWidthHeight()
      this.scaleWidth = min.w;
      this.scaleHeight = min.h;
      this.scaleX = -(this.scaleWidth - this.width) / 2
      this.scaleY = -(this.scaleHeight - this.height) / 2

      // 重置缩放控件的值
      this.scale = 100
    },
    getMinWidthHeight () {
      let w, h
      let r = this.img.naturalWidth / this.img.naturalHeight
      if (r >= 1) {
        h = this.height
        w = this.height * r
        if (w < this.width) {
          w = this.width
          h = this.width / r
        }
      } else {
        w = this.width
        h = this.width / r
        if (h < this.height) {
          h = this.height
          w = this.height * r
        }
      }
      return { w, h }
    },
    fileChange (e) {
      this.input = e.target
      if (e.target.files.length === 0) return false
      this.showCrop = true
      this.showCropDialog = false
      // 读取本地图片转成base64显示到页面待使用
      let fr = new window.FileReader()
      fr.onload = e => {
        this.img = new window.Image()
        this.img.src = fr.result
        this.img.onload = () => {
          this.scaleComponent.call(this)
          this.cropImage = fr.result
          this.showCropDialog = true
        }
      }
      fr.readAsDataURL(e.target.files[0])
    },
    cropDragBegin (e) {
      this.drag = true
      this.grabbing = true
      if (this.isPE) {
        this.beginX = e.touches[0].clientX - e.target.offsetLeft
        this.beginY = e.touches[0].clientY - e.target.offsetTop
        if (e.touches.length === 2) {
          this.beginLength = this.beginLength === 0 ? this.cropTouchData(e).length : this.beginLength
        }
        document.addEventListener('touchmove', this.cropDragMove, false)
        document.addEventListener('touchend', this.cropDragEnd, false)
      } else {
        this.beginX = e.x - e.target.offsetLeft
        this.beginY = e.y - e.target.offsetTop
        document.addEventListener('mouseover', this.cropDragMove, false)
        document.addEventListener('mouseup', this.cropDragEnd, false)
      }
    },
    cropDragMove (e) {
      if (!this.drag) return false
      // 放大
      if (this.isPE) {
        if (e.touches.length === 2) {
          this.scaleLength = this.cropTouchData(e).length
          this.scale = Math.min(200, Math.max(100, this.scaleLength / this.beginLength * 100))
          this.cropZoom()
        } else {
          this.scaleX = e.touches[0].clientX - this.beginX
          this.scaleY = e.touches[0].clientY - this.beginY
        }
      } else {
        this.scaleX = e.x - this.beginX
        this.scaleY = e.y - this.beginY
      }
      this.cropLimit()
    },
    cropDragEnd (e) {
      this.drag = false
      this.grabbing = false
      if (this.isPE) {
        document.removeEventListener('touchmove', this.cropDragMove, false)
        document.removeEventListener('touchend', this.cropDragEnd, false)
      } else {
        document.removeEventListener('mouseover', this.cropDragMove, false)
        document.removeEventListener('mouseup', this.cropDragEnd, false)
      }
    },
    cropToCanvas () {
      let min = this.getMinWidthHeight()
      // 最后剪出来的图片宽高
      let rw = Math.round(this.width / min.w * (100 / this.scale) * this.img.naturalWidth)
      let rh = Math.round(this.height / min.h * (100 / this.scale) * this.img.naturalHeight)
      this.canvas.width = rw
      this.canvas.height = rh
      this.context.clearRect(0, 0, rw, rh)
      // 在原图中要裁剪的部分
      let sx = Math.round(Math.abs(this.scaleX) * this.img.naturalWidth / this.scaleWidth)
      let sy = Math.round(Math.abs(this.scaleY) * this.img.naturalHeight / this.scaleHeight)
      let sw = Math.round((Math.abs(this.scaleX) + this.width) / this.scaleWidth * this.img.naturalWidth) - sx
      let sh = Math.round((Math.abs(this.scaleY) + this.height) / this.scaleHeight * this.img.naturalHeight) - sy
      this.context.drawImage(this.img, sx, sy, sw, sh, 0, 0, rw, rh)
    },
    cropZoom () {
      let s = this.scale / 100
      let sx = (Math.abs(this.scaleX) + this.width / 2) / this.scaleWidth
      let sy = (Math.abs(this.scaleY) + this.height / 2) / this.scaleHeight
      let min = this.getMinWidthHeight()
      this.scaleWidth = min.w * s
      this.scaleHeight = min.h * s
      this.scaleX = -sx * this.scaleWidth + this.width / 2
      this.scaleY = -sy * this.scaleHeight + this.height / 2
      this.cropLimit()
    },
    // 获取多点触控
    cropTouchData (e) {
      if (e.touches.length < 2) return
      let x1 = e.touches[0].pageX
      let x2 = e.touches[1].pageX
      let x3 = (x1 <= x2 ? (x2 - x1) / 2 + x1 : (x1 - x2) / 2 + x2)
      let y1 = e.touches[0].pageY
      let y2 = e.touches[1].pageY
      let y3 = (y1 <= y2 ? (y2 - y1) / 2 + y1 : (y1 - y2) / 2 + y2)
      return {
        length: Math.round(Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2))),
        x: Math.round(x3),
        y: Math.round(y3)
      }
    },
    // 限制边界
    cropLimit () {
      if (this.scaleX < -(this.scaleWidth - this.width)) this.scaleX = -(this.scaleWidth - this.width)
      if (this.scaleY < -(this.scaleHeight - this.height)) this.scaleY = -(this.scaleHeight - this.height)
      if (this.scaleX > 0) this.scaleX = 0
      if (this.scaleY > 0) this.scaleY = 0
      if (this.scaleHeight === this.height) this.scaleY = 0
      if (this.scaleWidth === this.width) this.scaleX = 0
    },
    // 提交裁剪
    cropOK () {
      // 将图像转为blob
      this.cropToCanvas()
      let data = this.canvas.toDataURL('image/jpeg', this.quality)
      data = window.atob(data.split(',')[1])
      let aBuffer = new ArrayBuffer(data.length)
      let uBuffer = new Uint8Array(aBuffer)
      for (let i = 0; i < data.length; i++) {
        uBuffer[i] = data.charCodeAt(i)
      }
      let blob = new window.Blob([uBuffer], { type: 'image/jpeg' })
      blob.filename = this.filename
      this.showCrop = false
      this.input.value = ''
      this.$emit('callback', blob)
    },
    // 取消裁剪
    cropCancel () {
      this.showCrop = false
      this.input.value = ''
    },
    cropGetRatio () {
      var ih = this.img.naturalHeight
      var canvas = document.createElement('canvas')
      canvas.width = 1
      canvas.height = ih
      var ctx = canvas.getContext('2d')
      ctx.drawImage(this.img, 0, 0)
      var data = ctx.getImageData(0, 0, 1, ih).data
      var sy = 0
      var ey = ih
      var py = ih
      while (py > sy) {
        var alpha = data[(py - 1) * 4 + 3]
        if (alpha === 0) {
          ey = py
        } else {
          sy = py
        }
        py = (ey + sy) >> 1
      }
      var ratio = (py / ih)
      return (ratio === 0) ? 1 : ratio
    }
  }
}
</script>
