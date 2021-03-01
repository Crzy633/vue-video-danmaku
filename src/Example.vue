<template>
  <div id="Example">
    <PhotoUpload :v-show="photo.show"
                 @click="photo.show = true"
                 @callback="HelloWorld"
                 :filename="photo.filename"
                 :maxWidth=480>
      上传照片
    </PhotoUpload>
    <img ref="b" />
  </div>
</template>

<script>
import PhotoUpload from './PhotoUpload'
export default {
  components: { PhotoUpload },
  data () {
    return {
      photo: {
        show: false,
        filename: 'abc.jpg',
      }
    }
  },
  methods: {
    // 点击"裁剪"之后会调用此方法，blob就是裁剪后的图片
    HelloWorld (blob) {
      // 可以把blob放进formdata上传到服务器
      console.log(blob)
      // 我这里演示,把blob转成Base64URL然后放在<img>上
      let a = new FileReader()
      a.onload = e => {
        this.$refs.b.src = e.target.result
      }
      a.readAsDataURL(blob)
    }
  }
}
</script>

<style lang="scss" scoped>
.photo {
  width: 160px;
  height: 90px;
  line-height: 90px;
  text-align: center;
  background: chocolate;
  border: 1px solid black;
  border-radius: 3px;
}
</style>