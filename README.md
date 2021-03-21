# vue-video-danmaku

基于vuejs2.0 + webpack + scss + element-ui环境使用的视频弹幕



演示链接：https://parva.cool/vue-video-danmaku/



# 使用方法

### (建议直接下载文件，看例子代码)

----

1. 将下载的两个文件(danmaku.vue、danmakuIcon.js)放进自己的项目中
2. import danmaku.vue并将其变成组件

```js
import danmaku from './danmaku'
...
components: { danmaku }
```

3. 使用组件

```html
<danmaku :data="danmaku"
         :src="videoUrl"
         @send="sendDanmaku"
         :width="1200"></danmaku>
```







# 属性

* data  (Array)			 弹幕资源
* src (String)				视频Url地址
* width(Number)		 [可选，默认600] 视频宽度，会按照16:9限制视频高度
* @send(Function)	  点击发送弹幕之后调用的方法，将弹幕发送到服务器







# 注意事项

1. 内嵌了Video标签，我自己自定义了组件，想添加什么功能可能需要修改源码（源码写得很乱。）







> 作者：Crzy633(Parva和Stupid Dragon)

