
#### 场景


今天忽然临时接到一个需求:
就是将markdown文件直接在vue项目中进行加载,并正常显示出来。
这......,我知道是可以进行加载markdown文件的。
但是我之前没有做过，答复的是:可以做的，但是这个需要一点时间。
领导:那行,你先调研一下。


#### 简单介绍 vue\-markdown\-loader


vue\-markdown\-loader可以将 Markdown 文件转换成Vue组件。
安装 npm i vue\-markdown\-loader \-D


#### 步骤1:在vue.config.js文件中去配置



```
module.exports = {
  chainWebpack:config=>{
    // 定义一个新的webpack模块规则，命名为md
    config.module.rule('md')
    // 通过.test()方法，指定这个规则应该匹配哪些文件
    // 这个规则将应用于所有以.md结尾的文件，即Markdown文件
      .test(/\.md/)
      // 使用vue-loader来处理Markdown文件
      .use('vue-loader')
      .loader('vue-loader')
      .end()
      // 指定vue-markdown-loader来处理Markdown文件
      .use('vue-markdown-loader')
      // 使用vue-markdown-loader包中的markdown-compiler模块来处理Markdown文件
      .loader('vue-markdown-loader/lib/markdown-compiler')
      // raw: true以原始字符串的形式处理Markdown内容，不进行HTML转义等处理。
      .options({
        raw: true
      })
  }
}

```

#### 哦豁\-项目启动报错


遇见的问题1:SyntaxError: Unexpected token '??\='
产生问题的原因：你的node版本是否太低。
在项目中验证是否支持??\=,可以验证一下。太低的话升级版本就行
还有一种可能：less\-loader或者sass\-loader或者其他的包的版本不对。


遇见的问题2: Syntax Error: TypeError: Cannot read property ‘styles‘ of undefined
产生问题的原因：vue\-loader的版本太高造成的。
我的项目是webpack的版本是:webpack5,它对应的vue\-loader应该是vue\-loader15,
我将它降级为：vue\-loader@15


#### 步骤2:在使用的页面



```
<template>
  <div>
    <showMarkdown>showMarkdown>
  div>
template>
<script>
// 引入的
import showMarkdown from './biji.md'
export default {
  components:{
    showMarkdown
  },
  data() {
    return {
    
    }
  }
}
script>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155511547-1597111656.png)


#### 发现问题:优化样式


我们需要下载 github\-markdown\-css
npm i github\-markdown\-css \-S
这个是用来优化markdown展示出来的样式
能够保持与GitHub相同的视觉效果
在需要的文件中引入 import 'github\-markdown\-css';
然后我们在组件的父级使用markdown\-body这个类来美化markdown



```
<template>
  <div>
    <div class="markdown-body">
      <showMarkdown>showMarkdown>
    div>
  div>
template>

<script>
import 'github-markdown-css';
import showMarkdown from './biji.md'
export default {
  components:{
    showMarkdown
  }
}
script>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155521098-68212827.jpg)


#### 可以把markdown文件在路由中引入吗？


有的小伙伴说:既然我们能够在页面中引入当成组件，
那可以在路由文件中引入嘛？
回答:可以的。下面我们就来看下



```
const routes = [
  {
    path: '/',
    name: 'Home',
    component: ()=>import("../views/echarts.vue")
  },
  {
    path: '/xuexi',
    name: 'xuexi',
    component: ()=>import("../views/xuexi.vue")
  },
  {
    path: '/md',
    name: 'md',
    // 引入的md文件
    component: ()=>import("../views/biji.md")
  },
]
const router = new VueRouter({
  mode: 'hash',
  base: process.env.BASE_URL,
  routes
})

```

#### main.js中引入 'github\-markdown\-css';



```
import Vue from 'vue'
import App from './App.vue'
import router from './router'
//全局引入
import 'github-markdown-css';
import ElementUI from 'element-ui';
import 'element-ui/lib/theme-chalk/index.css';
Vue.config.productionTip = false

Vue.use(ElementUI);
new Vue({
  router,
  render: h => h(App)
}).$mount('#app')

```

#### app.vue使用样式



```
<template>
  <div id="app" class="markdown-body">
    <div id="nav">
      <router-link to="/">Homerouter-link> |
      <router-link to="/about">Aboutrouter-link> |
      <router-link to="/about">Aboutrouter-link> |
      <router-link to="/echarts">echartsrouter-link> |
      <router-link to="/art">artrouter-link> |
      <router-link to="/test">testrouter-link> |
      <router-link to="/xuexi">xuexirouter-link> |
    div>
    <router-view/>
  div>
template>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155531530-1181679595.png)


#### 发现问题：markdown\-body 污染了全局样式


我们发现这样整个项目中都有 markdown\-body 这个类了。
这样会影响其他组件的布局样式。
我们只想在引入的文件是md才有这个样式。
其他的文件没有这个样式。
这个是否我们可以在app.vue文件中判断是否是md文件。
如果是md文件我们添加上markdown\-body这个类，否则移除。
我们在路由文件中的meta属性来判断是否是md文件类型


#### 路由文件



```
const routes = [
 {
    path: '/md',
    name: 'md',
    component: ()=>import("../views/biji.md"),
    meta:{
      fileType:'md'
    }
  },
  {
    path: '/amd',
    name: 'amd',
    component: ()=>import("../views/amd.md"),
    meta:{
      fileType:'md'
    }
  }
]

```

#### app.vue



```
<template>
  <div id="app" :class="componentPathName=='md' ? 'markdown-body' : null">
    <div id="nav">
      <router-link to="/">Homerouter-link> |
      <router-link to="/about">Aboutrouter-link> |
      <router-link to="/about">Aboutrouter-link> |
      <router-link to="/echarts">echartsrouter-link> |
      <router-link to="/art">artrouter-link> |
      <router-link to="/test">testrouter-link> |
      <router-link to="/xuexi">xuexirouter-link> |
    div>
    <router-view/>
  div>
template>
<script>
 export default {
  computed: {
    componentPathName () {
      return this.$route.meta && this.$route.meta.fileType
    }
  },
 }
script>

```

#### md文件内容有些时候是从服务端获取的


上面我们渲染的都是本地的文件.
如果 markdown 的内容是从服务端获取的。
动态渲染怎么去处理呢？
我们需要下载 vue\-markdown
npm install vue\-markdown \-\-save
然后在vue.config.js文件中去配置，与上面的配置相同(一样的哈)


#### vue\-markdown


它允许在Vue应用中轻松展示Markdown格式的内容。
它支持标准的Markdown语法。
如标题、列表、链接、图片、代码块等，并能够将Markdown文本解析为HTML格式。
从而在Vue组件中展示。


#### vue\-markdown 的简单使用



```
<template>
  <div>
    <VueMarkdown>
     {{ mdCont }}
    VueMarkdown>
  div>
template>

<script>
import VueMarkdown from 'vue-markdown';
export default {
  components:{
    VueMarkdown
  },
  data() {
    return{
      mdCont:'#### 绘制一个矩形的思路我们这里绘制矩形\n会使用到canvas.strokeRect(x,y, w, h)方法绘制一个描边矩形![](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/91e6190a5cdf4b548cbb7db766acb01c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5oiR55qEZGl25Lii5LqG6IK_5LmI5Yqe:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMTMxMDI3MzU5MzQ0MDM5OCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1733142556&x-orig-sign=Zghvt5lD2jYz6D0D0SaZye5cgos%3D)'
    }
  }
}
script>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155544955-1942416998.jpg)


#### 远端请求的内容为啥渲染失败



```
<template>
  <div>
    <VueMarkdown>
     {{ mdCont }}
    VueMarkdown>
  div>
template>

<script>
import VueMarkdown from 'vue-markdown';
export default {
  components:{
    VueMarkdown
  },
  data() {
    return{
      mdCont:'', //返回来的内容
      showKey: '0',
    }
  },
  created(){
    this.serveAPi()
  },
  methods:{
    serveAPi(){
      setTimeout(() => {
        this.mdCont = '#### 绘制一个矩形的思路我们这里绘制矩形\n会使用到canvas.strokeRect(x,y, w, h)方法绘制一个描边矩形![](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/91e6190a5cdf4b548cbb7db766acb01c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5oiR55qEZGl25Lii5LqG6IK_5LmI5Yqe:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMTMxMDI3MzU5MzQ0MDM5OCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1733142556&x-orig-sign=Zghvt5lD2jYz6D0D0SaZye5cgos%3D)'
        //更新设置这个key值
        this.showKey = new Date().getTime()+ ''
      },400)
    }
  },
}
script>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155552774-928459956.jpg)


我们发现md无法正常渲染，但是直接写在data中的是可以渲染的。
说明返回来的数据，在渲染的时候组件没有重新更新。
我们只需要使用key更新更新一下就行了。


#### 远端请求内容渲染markdown,key更新组件



```
<template>
  <div>
    
    <VueMarkdown :key="showKey">
     {{ mdCont }}
    VueMarkdown>
  div>
template>

<script>
import VueMarkdown from 'vue-markdown';
export default {
  components:{
    VueMarkdown
  },
  data() {
    return{
      mdCont:'', //返回来的内容
      showKey: '0',
    }
  },
  created(){
    this.serveAPi()
  },
  methods:{
    serveAPi(){
      setTimeout(() => {
        this.mdCont = '#### 绘制一个矩形的思路我们这里绘制矩形\n会使用到canvas.strokeRect(x,y, w, h)方法绘制一个描边矩形![](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/91e6190a5cdf4b548cbb7db766acb01c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5oiR55qEZGl25Lii5LqG6IK_5LmI5Yqe:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMTMxMDI3MzU5MzQ0MDM5OCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1733142556&x-orig-sign=Zghvt5lD2jYz6D0D0SaZye5cgos%3D)'
        //更新设置这个key值
        this.showKey = new Date().getTime()+ ''
      },400)
    }
  },
}
script>

```

![](https://img2024.cnblogs.com/blog/1425695/202411/1425695-20241126155603250-1849572129.jpg)


 本博客参考[楚门加速器](https://chuanggeye.com)。转载请注明出处！
