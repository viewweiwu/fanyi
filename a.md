# 【译】Vue + SVG，自由的控制 css 的动画详细解说。（附代码）

大家好，这里是 UX（交互体验设计师）& 前端开发，并且喜欢玩画画的 yuki![@yuneco][1]。 在这篇文章里，为了能够更加自由地使用 Vue 和 `css animation`，基础的部分，将会跟大家一步步的详细解释。目标如下图↓所示，用 `JavaScript` 自由的控制动画。

![效果图][2]

源码送上 [https://github.com/yuneco/css-anime-tutorial]( https://github.com/yuneco/css-anime-tutorial)

## 目录
那么先从最简单的 svg 标签开始。用 Vue 自由地配置、使其变形。然后利用 `css` 的 `transition` 来做动画，最后把动画抽象封装，运用到更加复杂的场景上。

  1. 制作 svg
  2. 创建 Vue 项目
  3. 显示 svg
  4. 能够自由配置
  5. 能够更自由更大角度的变化
  6. 赋予动画
  7. 能够连续进行动画
  8. 抽象封装动画

## 注意点
  1. 这篇文章介绍的方法并不是使用动画时通用的方法。
  2. 想要制作更复杂的动画，请使用 `anime.js` 或者 `pixi.js`。
  3. 这篇文章并没有使用专门的动画库，而是自己封装的动画，目标是为了更加深入地理解 Vue、`javascript`、`css 动画`。

虽然还有很多理由，但能点亮自己的【自己组建能够理解的动画】这方面的技能树，无疑也是很高兴的。文章稍稍有点长，如果你能看完那我也很高兴。

## 制作 svg
第一步，显示这篇教程要使用到的 svg，用 Illustrator 制作自己喜欢的角色去，依次从菜单上选择 [ファイル] -> [書き出し] -> [スクリーン用に書き出し]，格式选择 svg，从右边齿轮一样的图标，显示设定。

![制作svg][3]

设定看起来有点复杂的😓，这里在 Vue 也没有那么麻烦的去使
用 svg，所以这里的设定不需要太在意，右下角的 `Responsive(レスポンシブ)` 选择记得要取消掉。

设定好了之后，「設定を保存」->「アートボードを書き出し」导出 svg 文件，没有 `Illustrator` 的同学用其它的文件也 ok。怕麻烦的同学，我姑且在 [github][4] 上也放了一份...

用浏览器打开，大概就是这种感觉，名字叫 `taku`，现在刚决定的。为了方便理解，我加了 1 像素的边框。

![示例][5]

## 创建 Vue 项目
不管怎么说，不创建的 Vue 项目的话，就没法开始。运行，`vue create 项目的名字` 创建项目，就像下面一样，当然，你也可以根据自己的喜好来。
```
? Please pick a preset: 
  default (babel, eslint) 
❯ Manually select features 

? Check the features needed for your project: 
 ◉ Babel
 ◯ TypeScript
 ◯ Progressive Web App (PWA) Support
 ◯ Router
 ◯ Vuex
❯◉ CSS Pre-processors
 ◉ Linter / Formatter
 ◯ Unit Testing
 ◯ E2E Testing

? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules are supported 
by default): (Use arrow keys)
❯ Sass/SCSS (with dart-sass) 
  Sass/SCSS (with node-sass) 
  Less 
  Stylus 

? Pick a linter / formatter config: 
  ESLint with error prevention only 
  ESLint + Airbnb config 
❯ ESLint + Standard config 
  ESLint + Prettier 

之后全部默认
```

项目创建好了之后，删除掉多余的 `HelloWorld` 组件，然后在空项目里运行 `npm run serve` 确认可以跑起来。

## 显示 svg
刚刚做好的 svg 文件 放在 `/public/img/`, `img` 目录没有的话就创建一下。那就先用 Vue 把 svg 显示出来吧。

只是普通的显示 svg 的话跟 Vue 没有关系。另外为了之后的考虑，先把 `src/components/Tama.vue` 文件创建好，代码像下面这样。

``` html
<template>
  <img src="/img/tama.svg" alt="タマさん">
</template>
```

这篇教程用最简单的方法，用 `img` 标签来读取 svg。当然用其它的方法也行，就像下面这样：

  1. 直接使用 \<svg\> 标签
  2. 使用 css 的 background-image
  3. 使用 vue-svg-loader 之类的来导入成 Vue 组件

当然还有各种各样的方法，特别是方法 1，用起来特别爽(颜色和形状可以在 Vue 里面控制等等)，如果想做的复杂点，可以试试这个。
建好 `Tama.vue` 组件后呢，在 `App.vue` 里面引入显示。


``` html
<template>
  <div id="app">
    <tama></tama>
  </div>
</template>

<script>
import Tama from './components/Tama.vue'
export default {
  name: 'app',
  components: {
    Tama
  }
}
</script>

<style lang="scss">
html, body {
  margin: 0;
  padding: 0;
}
body {
  position: relative;
  height: 100%;
  background: url('/img/grid.svg') repeat;
}
#app {
  margin: 0;
}
</style>
```

这里只是把 Tama 导入而已。
清空了默认样式之后，为了方便展示，背景用 [网格][6] 展示。
展示成下面这样就 ok。

![示例][7]

## 配置到你想要的地方

画面已经出来了，如果不能自由的展示到想要的地方，就没法做动画。接下来就是把 `taku` 配置到任意的地方放。

### 配置到指定的坐标

先配置到到 `x = 200px, y = 100px` 这个位置吧。

``` html
<template>
  <img
    class="tama-root"
    src="/img/tama.svg"
    alt="タマさん"
  >
</template>

<style lang="scss" scoped>
.tama-root {
  position: absolute;
  left: 0;
  top: 0;
  transform: translate(200px, 100px);
}
</style>
```
用 `position: absolute` 绝对定位，然后用 `transform` 指定坐标，虽然指定位置也可以用 `top` 跟 `left`，但是 用 css 来做动画的时候，尽量还是用


[1]:https://twitter.com/yuneco
[2]:https://camo.qiitausercontent.com/267ab93e4087a9b5325f7601ff12fbbc87b10bd3/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f62323061313936652d666433392d666564332d623464632d6336363330616330663735342e676966
[3]:https://camo.qiitausercontent.com/f6e1e5e7b022f32d7ae2e0f24d08f4c1276a65b6/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f38626630316230392d366237342d393133312d613538632d3233353662653734613064352e706e67
[4]:https://github.com/yuneco/css-anime-tutorial/blob/master/public/img/tama.svg
[5]:https://camo.qiitausercontent.com/4c768a02fa1159b6aef70ef169bf3eec13020671/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f38343334323864392d313630652d386536372d383334312d3864663764386430346133372e706e67
[6]:https://github.com/yuneco/css-anime-tutorial/blob/master/public/img/grid.svg
[7]:https://camo.qiitausercontent.com/3a81f224dde487aca31e3e69b3984ad216a1a19d/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f32333835626165662d336630662d303763622d646665342d3465653163323362346636642e706e67