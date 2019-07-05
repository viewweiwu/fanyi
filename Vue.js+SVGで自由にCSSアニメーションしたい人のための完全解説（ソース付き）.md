#【译】Vue + svg，面向想要自由的驾驭 css 动画的人详细解说。（附代码）
大家好，这里是 UX（交互体验设计师）& 前端开发，并且还喜欢画画的 yuki！[@yuneco][1]。 在这篇文章里，为了能够更加轻松地去使用 Vue 和 `css animation`，基础的部分，将会跟大家一步步的详细解释。目标如下图↓所示，用 `JavaScript` 自由的控制动画。

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

虽然还有很多理由，但能点亮自己的【自己组建能够理解的动画】这方面的技能树，无疑也是很高兴的。文章稍稍有点长，如果你能看完的话，那我也很高兴。

## 制作 svg
第一步，显示这篇教程要使用到的 svg，用 Illustrator 制作自己喜欢的角色去，依次从菜单上选择 [ファイル] -> [書き出し] -> [スクリーン用に書き出し]，格式选择 svg，从右边齿轮一样的图标，显示设定。

![制作svg][3]

设定看起来有点复杂的😓，这里在 Vue 也没有那么麻烦的去使
用 svg，所以这里的设定不需要太在意，右下角的 `Responsive(レスポンシブ)` 选择记得要取消掉。

设定好了之后，「設定を保存」->「アートボードを書き出し」导出 svg 文件，没有 `Illustrator` 的同学用其它的文件也 ok。怕麻烦的同学，我姑且在 [github][4] 上也放了一份...

用浏览器打开，大概就是这种感觉，名字叫 `tama桑`，现在刚决定的。为了方便理解，我特意加了 1 像素的边框。

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

项目创建好了之后，删除掉多余的 `HelloWorld` 组件，然后在空项目里运行 `npm run serve` 确认可以跑起来就行。

## 显示 svg
刚刚做好的 svg 文件 放在 `/public/img/`, `img` 目录没有的话就自己建一下。接下来就先用 Vue 把 svg 显示出来吧。

只是普通的显示 svg 的话跟 Vue 没有关系。另外为了之后的考虑，先把 `src/components/Tama.vue` 文件创建好，代码像下面这样。

``` html
<template>
  <img src="/img/tama.svg" alt="タマさん">
</template>
```

这篇教程用最简单的方法，用 `img` 标签来读取 svg。当然用其它的方法也行，比如像下面这样：

  1. 直接使用 \<svg\> 标签
  2. 使用 css 的 background-image
  3. 使用 vue-svg-loader 之类的插件来导入成 Vue 组件

当然还有各种各样的方法，特别是方案 1，用起来特别爽(颜色和形状可以在 Vue 里面控制等等)，如果想做的复杂点，可以试试这个。建好 `Tama.vue` 组件后呢，在 `App.vue` 里面引入显示。

``` html
// Tama.vue
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

画面已经出来了，如果不能自由的展示到想要的地方，就没法做动画。接下来就是把 `tama桑` 配置到任意的地方放。

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
用 `position: absolute` 绝对定位，然后用 `transform` 指定坐标，虽然指定位置也可以用 `top` 跟 `left`，但是用 css 来做动画的时候，尽量还是用 transform 吧。只是指定元素位置的话，这样子做比较轻松(还有各种其它需求)，并且由 GUP 渲染的话，实现的动画能够丝般顺滑。

ok，完成！`tama桑` 的位置成功地挪动到了 `(200px, 100px)`！

![示例][8]

嗯？稍等😅，明明让 `tama桑` 的位置显示在 `(200px, 100px)` 的位置，为什么坐标是以左上角为基准，然后画也跟着到这个位置。还是想把她的脚底作为基准点。

![示例][9]

做法有很多，这次就用 margin 来调整吧。
``` css
.tama-root {
  // ...
  margin: -300px auto auto -90px;
}
```

![示例][10]

### 用参数来控制坐标

之前用的是定死的坐标，在实际使用时，还是会想要在变话坐标的时候加上动画的吧。所以当然不是在 css 写死的，用 Vue 的 props 来控制吧。

``` html
// Tama.vue
<script>
export default {
  name: 'Tama',
  props: {
    x: { type: Number, default: 200 },
    y: { type: Number, default: 100 }
  }
}
</script>
```

在 `Tama.vue` 里追加新代码，加上 `x, y` 两个参数，然后也把 type 和 default 指定下。
因为会用 poprs 来动态控制样式，所以在 \<style\> 把 transform 删掉。

```
// Tama.vue
.tama-root {
  // ...
  margin: -300px auto auto -90px;
  // 删掉: transform: translate(200px, 100px);
}
```

然后在 template 里指定要使用的 style。

``` html
// Tama.vue
<template>
  <img class="tama-root" src="/img/tama.svg" alt="タマさん"
    :style="{
      transform: `translate(${x}px, ${y}px)`
    }"
  >
</template>
```

然后在引用的地方 App.vue 里就可以指定 `tama桑` 的位置了。

``` html
// APP.vue
<div id="app">
  <tama :x="300" :y="400"></tama>
</div>
```

## 变化角度和大小
同理，我们来变化大小(scale)和角度吧。如果能够自由的控制位置、大小、角度，就能组合成自己想要画面。

### 用属性控制大小和角度
在相同的地方加上东西，scale 可以指定横向还是纵向，所以这里加上 `scaleX` 和 `scaleY` 两种属性。

``` javascript
// Tama.vue
props: {
  x: { type: Number, default: 200 },
  y: { type: Number, default: 100 },
  scaleX: { type: Number, default: 1.0 },
  scaleY: { type: Number, default: 1.0 },
  rotate: { type: Number, default: 0 }
}
```

然后让这些个属性在 template 里生效，scale、rotate 也是在 transfrom 里指定，很简单吧！scale 去掉单位， rotate 加上角度单位(deg)。

``` html
// Tama.vue
<img class="tama-root" src="/img/tama.svg" alt="タマさん"
  :style="{
    transform: `translate(${x}px, ${y}px) scale(${scaleX}, ${scaleY}) rotate(${rotate}deg)`
  }"
>
```

然后在引用的 App.vue 里指定下属性吧。

``` html
// App.vue
<div id="app">
  <tama :x="300" :y="400" :scale-x="1.5" :scale-y="1.5" :rotate="45"></tama>
</div>
```

跟指定位置一样很简单吧！嗯？等等...

![示例][11]

角度和大小确实是变化了，再就是基准点也有点奇怪😂

![示例][12]

为了解决这个问题，就用 css 的 `transform-origin` 属性吧。

``` css
// Tama.vue
.tama-root {
  // ...
  transform-origin: 90px 100%;
}
```

`transform-origin` 的单位可以是像素，也可以是百分号，然后我们来指定 `tama桑` 的位置吧。

![示例][13]

## 添加动画

终于... 做好了动画的准备工作，先来个简单的动画【单击向上跳 50px】，大概就只是【单击之后改变 `tama桑` y 轴的位置】。

### 单击改变位置

加动画既可以在 Tama.vue 里，也可以在 App.vue 里。这里想让 `tama桑` 自身有这个功能，所以在 Tama.vue 里做比较自然，就直接在这里写吧。

``` html
// Tama.vue
<template>
  <img class="tama-root" src="/img/tama.svg" alt="タマさん"
    :style="{...}"
    @click="jump(50)"
  >
</template>
...
<script>
export default {
  name: 'Tama',
  props: {...},
  methods: {
    jump (height) {
      this.y -= 50
    }
  }
}
</script>
```

用 `@click` 在单击时调用 jump 方法，在里面设定 y 为 -50，然后这样子做好了之后，单击 tama 就会向上移动 50px。

蛋疼的是，这个时候浏览器的 console 会有这样的警告。

![示例][14]

因为 y 是由父组件指定的，所以这里不能去改它，(x, y) 也就是指定它的位置而已，所以要在内部添加相对位置。

首先，为了要在内部添加变量，先把 data 写了，然后添加 dx、dy 两个变量。之后把 jump 方法里的 y 变成 dy 。

``` html
// Tama.vue
<script>
export default {
  name: 'Tama',
  props: {...},
  data () {
    return {
      dx: 0,
      dy: 0
    }
  },
  methods: {
    jump (height) {
      this.dy -= height
    }
  }
}
</script>
```

template 这边要指定坐标的话就是 = 基础位置(x, y) + 相对位置(dx, dy)。
``` html
// Tama.vue
<template>
  <img class="tama-root" src="/img/tama.svg" alt="タマさん"
    :style="{
      transform: `translate(${x + dx}px, ${y + dy}px) ...`
    }"
    @click="jump(50)"
  >
</template>
```

### 给变更位置加上动画

单击的时候位置确实变了，但是还没有动画。emm... 我们去加上动画吧。关于加动画的方法呢，我考虑到有下面两个点。

  1. 用连续的定时器控制 y 和 dy 坐标。
  2. 一口气设定好坐标，在 css 里做动画。

第 1 点虽然能够控制很复杂的动画，但是每次都要计算坐标，就会变的很麻烦。第 2 点的话，只要指定变更后的值，浏览器会帮我们加上动画，而且还很流畅。

这里当然选 2 啦。
代码就只要 2 行就够了。

``` css
// Tama.vue
.tama-root {
  // ...
  transition: transform 1s ease;
  will-change: transform;
}
```

`will-change` 是让动画变的更加顺滑的“魔法”。（本来用“魔法”来理解我觉得不太好，详细的我写了另一篇文章：[用 will-change 实现 60fps 动画][15]）

这样子做好之后，单击时 `tama桑` 就会滑上去了。简单的动画就这样实现好了( •̀ ω •́ )y。

![示例][16]

### 指定动画的时间和缓动效果

之前的例子我们指定好了动画的时间 (1s === 1秒)，和缓动效果 (ease) 这样子写死的代码。虽然这样子已经能够指定动画了，但我们也还是让她能够受这个的控制吧。 

data 里加上 duration 和 easing，template 里加上这两个变量。

``` html
// Tama.vue
<template>
  <img ...
    :style="{
      transform: ... ,
      transition: `transform ${duration}ms ${easing}`
    }"
    @click="jump(50)"
  >
</template>
...
<script>
export default {
  name: 'Tama',
  props: {...},
  data () {
    return {
      ...
      duration: 1000,
      easing: 'ease'
    }
  },
  ...
}
</script>
```

## 连续的动画（一个动画做完紧接着开始下一个动画）

跳当然是从地面上跳起来，再落下去。接下来要考虑下连续的动画。

大概就是这种感觉...

``` javascript
// Tama.vue
jump (height) {
  this.dy = -height
  // 等待上个动画结束
  this.dy = 0
}
```

### 用定时器实现连续的动画

我想了一下，等待的方法有下面两种：

  1. 单纯的用定时器等待
  2. 监听 transitionend 事件

虽然用 2 才是稳的，但是多个动画的话代码结构就会相当蛋疼，这次还是就用 1 吧。

``` javascript
// Tama.vue
jump (height) {
  this.dy = -height
  this.easing = 'ease-out'
  window.setTimeout(() => {
    this.dy = 0
    this.easing = 'ease-in'
  }, this.duration)
}
```

easing 也指定了之后，跳起来感觉就会像云一样轻飘飘的。跳完也会回到原来的位置。

![示例][17]

### 用 async/await 实现连续动画

之前，如果我们想要做复杂的动画，要是真的实现起来，那真的是... 一个字：吐血。

☠回调地狱（callback hell）☠警告。

如果要实现 3、4 个连续的动画，会让你知道什么叫不忍直视。

为了摆脱回调地狱，还是用上 async/await 吧。话虽如此，其实也没啥特别需要做的，只是把定时器 promise 化了而已。

``` javascript
// src/core/Time.js
export default {
  /**
  * Promise 等待指定时间
  * @param {Number} ms 等待时间
  * @return {Promise} 经过指定时间后 resolve
  */
  wait (ms) {
    return new Promise(resolve => {
      window.setTimeout(resolve, ms)
    })
  }
}
```

这里没有见过这样写的同学，可能理解起来会有点困难，这里的计时器 promise 化了之后，就可以像这样去控制等待时间。

``` javascript
console.log('这条消息显示了')
await Time.wait(2000) // 这里等待 2 秒
console.log('这条消息 2 秒后显示了')
```

这篇文章不会讲 Promise，也不会说明 async/await，记住用法的话，也就没啥问题了吧。

用上了 `Time.js` 之后，刚才那令人吐血的代码就可以写成下面这样子了：

``` html
// Tama.vue
<script>
import Time from '@/core/Time'
export default {
...

  async jump (height) {
    this.dy = -height
    this.easing = 'ease-out'
    await Time.wait(this.duration)
    this.dy = 0
    this.easing = 'ease-in'
    await Time.wait(this.duration)
  }

...
}
```

## 动画的抽象与封装

这样子，3个以上的动画也能够组合起来，连续的执行了。既然都到这一步了，干脆把它封装成像动画库一样吧。

### 封装成 Tween 风格

之前的例子，以下两个步骤会频繁的去操作动画帧：

  1. 频繁变更 data 里面的变量
  2. 用 `Time.wait` 等待动画结束

下面就是抽出来的另一种方法

``` javascript
Tama.vue
methods: {
  async tween (props, duration = 1000) {
    Object.assign(this.$data, props)
    this.$data.duration = duration
    await Time.wait(duration)
  },
  async jump (height) {
    await this.tween({ dy: -height, easing: 'ease-out' }, 1000)
    await this.tween({ dy: 0, easing: 'ease-in' }, 1000)
  }
}
```

刚追加的 tween 方法里用了 Object.assign，用参数传递的方式，把 props 覆盖掉 this.$data 的变量。之后再用 Time.wait 控制等待 duration 毫秒。
调用的时候要是用上这个方法，代码就会简约很多。

### 复杂动画的封装

封装好了 tween 方法之后呢，多个动画的组合就会变得很容易，用上这个让我再提炼下 jump 的代码吧。

``` javascript
// Tama.vue
async jump (height = 200, duration = 2500) {
  await this.tween({ dScaleY: 0.8, easing: 'ease' }, duration * 0.1)
  await this.tween({ dy: -height, dScaleY: 1.1, easing: 'ease-out' }, duration * 0.35)
  await this.tween({ dy: 0, dScaleY: 1.2, easing: 'ease-in' }, duration * 0.35)
  await this.tween({ dScaleY: 0.7, easing: 'ease' }, duration * 0.1)
  await this.tween({ dScaleY: 1.0, easing: 'ease' }, duration * 0.1)
}
```

跳跃前加上滞留的动作，跳跃的时候拉伸下身体，就算是只有一张图片，也可以做到 pióng pióng 像布丁一样很可爱的动作。😊

![示例][18]

到这里，总结一下，把代码全部都一口气贴出来。
另外，除了 jump，顺便把 walk 方法也给写了一下。

``` html
// Tama.vue
<template>
  <img class="tama-root" src="/img/tama.svg" alt="タマさん"
    :style="{
      transform: `translate(${x + dx}px, ${y + dy}px) scale(${scaleX * dScaleX}, ${scaleY * dScaleY}) rotate(${rotate + dRotate}deg)`,
      transition: `transform ${duration}ms ${easing}`
    }"
    @click="jump(200)"
  >
</template>

<style lang="scss" scoped>
.tama-root {
  position: absolute;
  left: 0;
  top: 0;
  margin: -300px auto auto -90px;
  transform-origin: 90px 100%;
  will-change: transform;
}
</style>

<script>
import Time from '@/core/Time'
export default {
  name: 'Tama',
  props: {
    x: { type: Number, default: 200 },
    y: { type: Number, default: 100 },
    scaleX: { type: Number, default: 1.0 },
    scaleY: { type: Number, default: 1.0 },
    rotate: { type: Number, default: 0 }
  },
  data () {
    return {
      dx: 0,
      dy: 0,
      dScaleX: 1.0,
      dScaleY: 1.0,
      dRotate: 0,
      duration: 1000,
      easing: 'ease'
    }
  },
  methods: {
    async tween (props = {}, duration = 1000) {
      Object.assign(this.$data, props)
      this.$data.duration = duration
      await Time.wait(duration)
    },
    async jump (height = 200, duration = 2500) {
      await this.tween({ dScaleY: 0.8, easing: 'ease' }, duration * 0.1)
      await this.tween({ dy: -height, dScaleY: 1.1, easing: 'ease-out' }, duration * 0.35)
      await this.tween({ dy: 0, dScale: 1.2, easing: 'ease-in' }, duration * 0.35)
      await this.tween({ dScaleY: 0.7, easing: 'ease' }, duration * 0.1)
      await this.tween({ dScaleY: 1.0, easing: 'ease' }, duration * 0.1)
    },
    async walk (step = 100, duration = 500) {
      await this.to({ dRotate: 10, dScaleY: 0.8, easing: 'ease' }, duration * 0.2)
      await this.to({ dx: this.dx + step, dy: -step * 0.2, dRotate: -5, dScaleY: 1.1, easing: 'cubic-bezier(.04,.67,.52,1)' }, duration * 0.7)
      await this.to({ dy: 0, dRotate: 0, dScaleY: 1, easing: 'ease' }, duration * 0.1)
    }
  }
}
</script>
```

### 组合动画

到这一步，就可以调用抽象好的，好几个动画组合成的，复杂的，那个叫做 jump 和 walk 方法。最后，再用这些方法去组合更加复杂的动画吧。

在 App.vue 加一个按钮，点击这个按钮，就可以给 `tama桑` 加上由 jump 和 walk 方法组合成的，连续的动画。

``` html
// App.vue
<template>
  <div id="app">
    <button @click="play">Play</button>
    <tama ref="tama" :x="100" :y="300" :scaleX="0.5" :scaleY="0.5"></tama>
  </div>
</template>
```

按下按钮的方法 play 大概就是这样的感觉:

``` javascript
// App.vue
async play () {
  const tama = this.$refs.tama
  await tama.jump(100, 1500)
  await tama.walk(100, 1200)
  await tama.walk(60, 600)
  await tama.walk(40, 400)
  await tama.jump(200, 2500)
}
```

小跳一下 -> 走 3 步 -> 最后来个大跳。这么一连续的动作，就可以这么简简单单的就能表示出来。这次只有 `tama桑` 一个人，如果再来多几个组合好动画的人物的话，应该就可以像游戏一样，组合成复杂的动作。

## 总结

  - css transform 的坐标，在 Vue 里用以参数的形式让它联动，就可以很轻松的配置 svg 的位置、大小、角度哦！
  - 用上 css transition 了的话，就可以给位置、大小、角度加上动画哦！
  - 用上 async/await 了的话，使用起来感觉就像 Tween 库一样，很复杂的动画也可以很轻松的去控制。

  1. 刚开始的时候决定好大小的话，用起来就很方便。
  2. Vue 的基本操作没有介绍，不理解的时候，适当的去参照 Vue 入门文章之类的。
  3. 之前也写过类文章，一个是用了 Vue 和 svg，做的射击游戏《<猫🐱鱼🐟攻击🌟>的代码解说》，那里面也用到了 `tama桑`，二是《用 Vue 和 firebase 的基本功能，制作流畅的个人网站代码解说》，里面的动画跟这篇文章是一样的。

## 译者记

动画看起来效果不错，文章写的非常的详细，基本上只要会一点 Vue，就可以轻松的驾驭了。不过 async/await 第一次见的话，那这里也能够体验一下感受，同步的写代码也是非常爽的，不过都 9102 年了，还没接触 async/await 的话就有点... 这篇文章是从日语技术网站 qiita 上面搬运过来的，所以会有一些日文，已经得到原作者的许可，之后的话看有机会继续翻译其它的吧。还有 `tama桑` 很可爱。

原地址 https://qiita.com/yuneco/items/1f31418269f9df15cf9f


[1]:https://twitter.com/yuneco
[2]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5ecdfb30a?w=600&h=255&f=gif&s=1115815
[3]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5d70c10fe?w=924&h=462&f=png&s=88528
[4]:https://github.com/yuneco/css-anime-tutorial/blob/master/public/img/tama.svg
[5]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5d6023b7c?w=802&h=696&f=png&s=72167
[6]:https://github.com/yuneco/css-anime-tutorial/blob/master/public/img/grid.svg
[7]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d76054e51f?w=902&h=762&f=png&s=67502
[8]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d76a856068?w=692&h=738&f=png&s=54866
[9]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5d4b40111?w=768&h=778&f=png&s=75196
[10]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5e25c7de4?w=544&h=338&f=png&s=24282
[11]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d5f62bfa7a?w=862&h=816&f=png&s=85316
[12]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d6e9fb429c?w=1396&h=688&f=png&s=203978
[13]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d6e415a0fd?w=1056&h=784&f=png&s=88703
[14]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d6d6519e21?w=892&h=230&f=png&s=50449
[15]:https://qiita.com/yuneco/items/f07d60ab21dd1de9a51d
[16]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d76698652e?w=400&h=317&f=gif&s=637393
[17]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d733550325?w=400&h=316&f=gif&s=405325
[18]:https://user-gold-cdn.xitu.io/2019/7/5/16bc11d6ffbf1ece?w=400&h=300&f=gif&s=408363