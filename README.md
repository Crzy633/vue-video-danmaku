# vue-photo-upload

基于vuejs2.0 + webpack + scss环境使用的上传组件

* 支持自定义图片比例
* 支持PC端和移动端，包括移动端手势支持
* 支持修改图片质量
* 支持裁减图片缩放
* 支持自定义方法
* 界面会自适应屏幕大小，支持限制最大宽高



<img src="https://stupid-dragon.oss-cn-beijing.aliyuncs.com/vue-photo-update/show.gif" alt="show.gif" style="zoom:80%;" />







# 使用方法

1. 将PhotoUpload.vue放进自己的项目中
2. import并将其变成组件

```javascript
import PhotoUpload from './PhotoUpload'
...
components: { PhotoUpload }
```

3. 设置样式（自己随意定义，这里的效果<img src="https://stupid-dragon.oss-cn-beijing.aliyuncs.com/vue-photo-update/button.png">）

```scss
PhotoUpload {
  width: 160px;
  height: 90px;
  line-height: 90px;
  text-align: center;
  background: chocolate;
  border: 1px solid black;
  border-radius: 3px;
}
```

4. 添加组件

```vue
<template>
  <div id="Example">
    <PhotoUpload :v-show="photo.show"
                 @click="photo.show = true"
                 @callback="HelloWorld"
                 :filename="photo.filename">
      上传照片
    </PhotoUpload>
  </div>
</template>
```

5. 设置属性

```javascript
data () {
  return {
    photo: {
      show: false,
      filename: 'abc.jpg'
    }
  }
},
methods: {
  // 点击"裁剪"之后会调用此方法，blob就是裁剪后的图片
  HelloWorld (blob) {
    // 可以把blob放进formdata上传到服务器
    console.log(blob.filename)
  }
}
```







# 可选属性

* :filename  (string)	文件名，会被添加到blob.filename中，例"abc.jpg"
* :ext  (string) 			支持格式，例"jpg,gif,png"
* :limit  (int)				限制大小
* :quality (int)			 降低图片质量，范围0~1，默认1
* :ratio (string)			图片比例，格式如"16:9"，默认不填的话会出现三个比例按钮
* :maxWidth(int)       限制该组件界面的宽度显示，默认350
* @callback (Function)   点击裁剪之后的回调







# 注意事项

1. 组件css样式中不要有transform属性，不然子组件的position:fixed会失效，变成absolute，导致整个界面错位
2. ext属性只支持组件内定义好的mimes属性，当然，有需要时自己可以随意修改组件对应源码
3. 填写了ratio属性之后，3个''比例按钮''会消失
4. 改造自[jinzhe](https://github.com/jinzhe/vue-upload)







> 作者：Crzy633(Parva和Stupid Dragon)