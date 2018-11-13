### 概述
工程化是我们长期以来对工作的积累的产物，帮助我们更快、更好、更高效的完成多样化的工作，我们工程化的主要目的在于：
+ 降低学习小程序开发门槛；
+ 提高代码的复用性；

### 框架选择
#### 1. mpvue

mpvue 是美团点评开源的一个使用 Vue.js 开发小程序的前端框架。框架基于 Vue.js 核心，mpvue 修改了 Vue.js 的 runtime 和 compiler 实现，使其可以运行在小程序环境中，从而为小程序开发引入了整套 Vue.js 开发体验。使用 mpvue 开发小程序，你将在小程序技术体系的基础上获取到这样一些能力：
+ 彻底的组件化开发能力：提高代码复用性
+ 完整的 Vue.js 开发体验
+ 方便的 Vuex 数据管理方案：方便构建复杂应用
+ 快捷的 webpack 构建机制：自定义构建策略、开发阶段 hotReload
+ 支持使用 npm 外部依赖
+ 使用 Vue.js 命令行工具 vue-cli 快速初始化项目
+ H5 代码转换编译成小程序目标代码的能力
+ 在未来最理想的状态是，可以一套代码可以直接跑在多端：WEB、小程序（微信和支付宝）、Native（借助weex）

#### 2. wepy

WePY 是一款让小程序支持组件化开发的框架，通过预编译的手段让开发者可以选择自己喜欢的开发风格去开发小程序。框架的细节优化，Promise，Async Functions 的引入都是为了能让开发小程序项目变得更加简单，高效。
特性：

+ 类 Vue 开发风格
+ 支持自定义组件开发
+ 支持引入 NPM 包
+ 支持 Promise
+ 支持 ES2015 + 特性，如 Async Functions
+ 支持多种编译器，Less/Sass/Styus、Babel/Typescript、Pug
+ 支持多种插件处理，文件压缩，图片压缩，内容替换等
+ 支持 Sourcemap，ESLint 等
+ 小程序细节优化，如请求列队，事件优化等
#### 3. 特性对比图
![特性对比图](small-programs.jpeg)
根据上图对比`mpVue`是我们最佳选择

### mpvue注意事项
##### 不支持 纯-HTML
程序里所有的 `BOM／DOM` 都不能用，也就是说 `v-html` 指令不能用。
##### 不支持部分复杂的 JavaScript 渲染表达式
我们会把 `template` 中的 `{{}}` 双花括号的部分，直接编码到 wxml 文件中，由于微信小程序的能力限制(数据绑定)，所以无法支持复杂的 JavaScript 表达式。

目前可以使用的有` + - * % ?: ! == === > < [] .`，剩下的还待完善。
```html
<!-- 这种就不支持，建议写 computed -->
<p>{{ message.split('').reverse().join('') }}</p>

<!-- 但写在 @event 里面的表达式是都支持的，因为这部分的计算放在了 vdom 里面 -->
<ul>
    <li v-for="item in list">
        <div @click="clickHandle(item, index, $event)">{{ item.value }}</p>
    </li>
</ul>
```
##### 不支持过滤器
渲染部分会转成 wxml ，wxml 不支持过滤器，所以这部分功能不支持。
##### 不支持函数
不支持在 template 内使用 methods 中的函数。
##### Class 与 Style 绑定
为节约性能，我们将 Class 与 Style 的表达式通过 compiler 硬编码到 wxml 中，支持语法和转换效果如下：

class 支持的语法:
```html
<p :class="{ active: isActive }">111</p>
<p class="static" v-bind:class="{ active: isActive, 'text-danger': hasError }">222</p>
<p class="static" :class="[activeClass, errorClass]">333</p>
<p class="static" v-bind:class="[isActive ? activeClass : '', errorClass]">444</p>
<p class="static" v-bind:class="[{ active: isActive }, errorClass]">555</p>
```
将分别被转换成:
```html
<view class="_p {{[isActive ? 'active' : '']}}">111</view>
<view class="_p static {{[isActive ? 'active' : '', hasError ? 'text-danger' : '']}}">222</view>
<view class="_p static {{[activeClass, errorClass]}}">333</view>
<view class="_p static {{[isActive ? activeClass : '', errorClass]}}">444</view>
<view class="_p static {{[[isActive ? 'active' : ''], errorClass]}}">555</view>
```
style 支持的语法:
```html
<p v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }">666</p>
<p v-bind:style="[{ color: activeColor, fontSize: fontSize + 'px' }]">777</p>
```
将分别被转换成:
```html
<view class="_p" style=" {{'color:' + activeColor + ';' + 'font-size:' + fontSize + 'px' + ';'}}">666</view>
<view class="_p" style=" {{'color:' + activeColor + ';' + 'font-size:' + fontSize + 'px' + ';'}}">777</view>
```
不支持 官方文档：Class 与 Style 绑定 中的 classObject 和 styleObject 语法。

最佳实践见上文支持的语法，从性能考虑，建议不要过度依赖此。

此外还可以用 computed 方法生成 class 或者 style 字符串，插入到页面中，举例说明：
```html
<template>
    <!-- 支持 -->
    <div class="container" :class="computedClassStr"></div>
    <div class="container" :class="{active: isActive}"></div>

    <!-- 不支持 -->
    <div class="container" :class="computedClassObject"></div>
</template>
<script>
    export default {
        data () {
            return {
                isActive: true
            }
        },
        computed: {
            computedClassStr () {
                return this.isActive ? 'active' : ''
            },
            computedClassObject () {
                return { active: this.isActive }
            }
        }
    }
</script>
```
[更多请查看使用手册](http://mpvue.com/mpvue/#_1)
### 使用第三方VantUI组件
#### 如何使用
修改src/pages.js
```js
{
path: 'pages/index/index',
config: {
  navigationBarTitleText: '乡村销客',
  enablePullDownRefresh: true,
  usingComponents: {
    'van-button': '/static/vant/button/index'
  }
}
```
index.vue
```html
<van-button>测试</van-button>
```
> 更多组件使用请查看[Vant官方文档](https://youzan.github.io/vant-weapp/#/intro)
#### 注意事项
##### 数据绑定
```js
v-bind:value="value"
//或者
:value="value"
```
##### 事件监听
```js
@click="onClick"
```
##### vue 中组件引入
vant中像notify这种操作反馈类的组件都有两个引入，一是组件的引入，这个在pages.js中引入；另一个是方法的引入，需要在vue文件中import引入，值得注意的是，这里的引入不能使用绝对路径，可以用类似于这样的相对路径。
```js
import Notify from '@/../static/notify/notify' //@是mpvue的一个别名，指向src目录
```
##### 获取 event
值得注意的是，mpvue中获取event值与原生小程序有所不同。举例：
```js
onChange(event){ // 获取表单组件filed的值
  console.log(event.mp.detail) // 注意加入mp
}
```
#### BUG 及报错处理方法
##### 监听名
mpvue 里面无法使用@click-icon这样的监听名,因此如果 API 文档里面有出现这样的监听名，那么需要手动修改源代码。
可以改成驼峰式的监听名。
```js
// static/vant/field/index.js

this.$emit('click-icon');

// 修改为:

this.$emit('clickIcon');
```
##### 报错

一般的报错报错都可以通过一下流程处理。

+ 是否打开了微信开发者工具中的ES6转ES5功能。
+ 仔细检查代码和比对文档，看看是否有使用不当的地方。
+ 重新编译npm start或删掉dist目录重新npm start
+ 重启或更新微信开发者工具。
若以上流程都走完了，还是无法解决报错，可以通过提交issues的方式，我来帮你解决。
##### 引入组件报错
```js
VM54:1 thirdScriptError sdk uncaught third Error module "static/vant/notify/index.js" is not defined
```
解决办法是：打开小程序开发者工具中的ES6 转 ES5功能
### 开发涉及技术
#### 图片
图片来源接口 [详细文档](https://developers.weixin.qq.com/miniprogram/dev/api/media/image/wx.chooseImage.html)
```js
wx.chooseImage({
  count: 1, // 默认9
  sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
  sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
  success (res) {
    // 返回选定照片的本地文件路径列表，tempFilePath可以作为img标签的src属性显示图片res.tempFilePaths
      
  },
  fail (res) {
    console.log(res.errMsg)
  }
})
```
图片上传阿里OSS
```js
var nowTime = formatTime(new Date())
// 一次上传多张
for (var i = 0; i < res.tempFilePaths.length; i++) {
// 显示消息提示框
  wx.showLoading({
    title: '上传中' + (i + 1) + '/' + res.tempFilePaths.length,
    mask: true
  })
  // 上传图片
  // 你的域名下的/cbb文件下的/当前年月日文件下的/图片.png
  // 图片路径可自行修改
  uploadImage.uploadFile(res.tempFilePaths[i], 'cbb/' + nowTime + '/', 
  function (result) {
    // 成功的操作
    console.log('======上传成功图片地址为：', result)
    wx.hideLoading()
  }, function (result) {
    console.log('======上传失败======', result)
    wx.hideLoading()
  })
}
```
图片放大 [详细文档](https://developers.weixin.qq.com/miniprogram/dev/api/media/image/wx.previewImage.html)
``` js
wx.previewImage({
  current: curPath, // 当前显示图片的http链接
  urls: this.tempFilePaths // 需要预览的图片http链接列表
})
```
#### 位置
使用经纬度获取详细地址 [详细文档](http://lbsyun.baidu.com/index.php?title=webapi/guide/webservice-geocoding-abroad)
```js
import commonAPI from '@/api/commonAPI'

commonAPI.getCurrentAddress(res.latitude, res.longitude).then(function (result) {
  // 处理获取来的数据
  var address = JSON.parse(result.split('renderReverse&&renderReverse(').join('').split(')').join(''))
  //输出详细地址
  console.log(address.result.formatted_address)
}).catch(function (err) {
  console.log(err)
})
```
获取经纬度接口 [详细文档](https://developers.weixin.qq.com/miniprogram/dev/api/location/wx.getLocation.html)
```js
wx.getLocation({
  type: 'gcj02',
  success (res) {
    // 可以输出的信息
    // const latitude = res.latitude
    // const longitude = res.longitude
    // const speed = res.speed
    // const accuracy = res.accuracy
  }
})
```
#### mpvue基本用法-js
```js
export default {
  // 数据定义
  data () {
    return {
      motto: 'Hello World',
      userInfo: {},
      index: 0,
      array: ['A', 'B', 'C']
    }
  },
  // 组件调用
  components: {
    ccamers
  },
  // 方法
  methods: {
    bindPickerChange (e) {
      // 一个普通的方法
      console.log(e)
      this.index = e.mp.detail.value
    },
    clickHandle (msg, ev) {
      // 点击后的回调函数
      console.log('clickHandle:', msg, ev)
    }
  },
  created () {
    // 初始化
    // 调用应用实例的方法获取全局数据
  }
}
```
