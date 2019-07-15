# 【译】使用 will-change 目标让浏览器拥有 60fps 如丝般流畅的 css 动画

大家好、这里就是用了大量动画去做个人主页、还有射击游戏的 `yuki`。

感觉 css 的动画用的太多了，我的 MacBook 都要“罢工”了，回过头来，我就给大家总结一下，我的实验结果吧。(还是好想用 css 动画

## 这次的例子

这篇文章的目的大概就是，在网页里用 css 动画加一点复杂的动画，然后去做一些游戏或者艺术表现的时候，全部都用上 css 的 will-change 属性，怎么样才能让 GPU 渲染最合适。

如果你要问 will-change 是什么？那可以看一下我放在这篇文章里的参考列表。

![示例][1]

https://css-anime.firebaseapp.com/

本次要验证的动画有下面几个点：

  - 用 css 画 1000 朵花❀，一朵一朵的做开画的动画
  - 每朵花都是 div 元素（这次不用 svg）
  - 每次间隔 32ms 画一朵
  - 开完之后的花，就保持停留在画面上
  - 并且画面一直会在旋转

不知道大家有没有遇到过，需要一边做动画，还得一直加标签，然后还得一直显示，且不能从画面消失，这种很痛苦的情况。

实验环境如下：

  - MacBookPro 2017Late / 8GB RAM（经常被叫做“梅”的最低配置）
  - MacOSX Mojave
  - Chrome 74

之后会提一下 Safari 下的情况。
因为不同的环境会有很大的差异，浏览器和操作系统不一样的话，结果会有差异，我先在这里事先说明一下。

## 站在巨人的肩膀：我觉得值得一读的文章

```!
译者注：下面链接全部都是*日文*的，标题我翻译了一下，日文 ok 的同学可以点点看...
```

这篇文章参考以下链接：

  - [CSS Will Change Module Level 1（日文版）][2]
  - [CSS: will-change 对性能的影响（日文版）][3]
  - [高性能 60fps 的 CSS3 动画最佳实践（日文版）][4]
  - [优先使用合成专用的属性，以及层级数的管理（日文版）][5]

## 实验0：基本元素与动画的构建

为了下手方便，我就先随便的写了一下，我把代码贴也上来了，大家看到代码应该也就就明白了。

花❀的话 html 部分大概就是这样的，为了轻松和个人兴趣，这里用了下 Vue，其实什么框架都是可以的。

`Flower.vue`
``` html
<div class="flower-root" 
  :class="{animate: visible}"
  @animationend="onEndAnim">
  <!-- 花瓣 -->
  <div class="petal" v-for="(petal, index) in petals" :key="index"
    :style="{
      transform: `rotate(${petal.r}deg)`,
      'background-color': petal.col
    }">
  </div>
  <!-- 中间 -->
  <div class="center"></div>
</div>
```

花瓣是用 div 搭的，除了角度和颜色是在 template 里面写的，其它的都是在下面 \<style\> 标签里写的。开花的动画也在这里写了（也就是说，动画跟 Vue 和 javascirpt 没有关系，就只是用 css 写的而已）。

`Flower.vue`
``` css
<style lang="scss" scoped>
  .flower-root {
    position: absolute;
    transform: rotate(0deg) scale(0);
    animation: rotate 2s ease-out 0s 1 normal forwards;
  }
  .petal {
    position: absolute;
    width: 70px;
    height: 20px;
    top: -10px;
    left: 0;
    transform-origin: left center;
    border-radius: 50px;
    background-color: #ffb7aa;
  }
  .center {
    position: absolute;
    width: 30px;
    height: 30px;
    left: -15px;
    top: -15px;
    border-radius: 30px;
    background-color: #ffe683;
  }
  // 一边旋转一遍变大
  @keyframes rotate {
    0% { transform: rotate(0deg) scale(0); }
    100% { transform: rotate(360deg) scale(1); }
  }
</style>
```

写出来的花大概就是这种感觉。

![示例][6]

让这个 Flower 组件每隔一定时间，就往画面里加，然后画面的整体让它转起来。

## 实验1：直接跑起来（没有写 will-change）

先试试不用 will-change，跑一下动画，打开 Chrome 的 performance monitor 面板。

![示例][7]

```!
译者注：可以 f12 -> ctrl + shift + p 输入 rendering 打开 fps 面板，图片上左侧的 gpu 面板是 mac 系统自带的。
```

CPU 还挺给力的，还保持着 60 fps，哦哟，好像还不错哟。

![示例][8]

500 个了。差不多 400 个左右的时候 CPU 已经到达瓶颈了，帧数一下子就掉下来了，浏览器上了 GPU，但是帧数好像还是很不稳定。

![示例][9]

最后，平均帧数到了差不多 20fps，风扇一直转的大声...

![示例][10]

所有的花都开完了，只剩下 1000 朵静止的花在旋转。到这一步，终于回复了 60fps。

### 实验 1 结论

  - css 动画会因为元素的比例上升，CPU 负荷会上涨。
  - CPU 到达瓶颈的时候，帧数会暴跌。


## 实验2：全部都用上 will-change

但是不试试看怎么会知道呢？变更的地方就是在 style 里加上 will-change: transform。

`Flower.vue`

``` css
.flower-root {
  position: absolute;
  transform: rotate(0deg) scale(0);
  animation: rotate 2s ease-out 0s 1 normal forwards;
  will-change: transform; // 追加
}
```

Let's start!

![示例][11]

第 100 个，非常的流畅，CPU 负荷也很低。

![示例][12]

500 个，开始有点挣扎了，GPU 就像吃了芥末一样，一下子就窜起来了，CPU 也有点负荷了。

![示例][13]

快到结尾了，CPU 和 GPU 都在疯狂的“惨叫”，performance 也是很危险的状态。

![示例][14]

所有的开花的动画都结束了之后，负荷也不会降下来， 写了 will-change 的话，浏览器为了保证接下来的随时会开始的动画的性能，就算动画结束了，负荷也不会下降。

### 实验 2 结论

  - 写了 will-change 的话，就会启动 GPU 加速
  - 带 will-change 的元素过多的话，GPU 和 CPU 都会加重负担
  - 写了 will-change 的元素，就算动画结束了，仍然不会减轻负担

## 实验3：动画结束后，删除 will-change 样式

```!
译者注：把 will-change 属性删除是指把 will-change 的指设置成默认的 auto，之后的翻译也都是如此。
```

实验 2 的问题是，开花的动画明明都已经结束了，但是 will-change 却存在着。因此，在实验 3，开花的动画结束后，就把 will-change 样式删除。

为了能够动态的设置 will-change 属性，所以把这个属性挪到 template 里了，然后加个变量去控制它（在 Vue 加了个 isMoving 变量）。监听 animationend 事件，在这个事件内，去改 isMoving 变量。

`Flower.vue`
``` html
<div class="flower-root" 
    :style="{
      'will-change': isMoving ? 'transform' : 'auto',
    }"
    @animationend="onEndAnim">
    ...
```

`Flower.vue`
``` javascript
private onEndAnim() {
  this.isMoving = false
}
```

检测到动画结束，只把 will-change 的值删除。

那么，start！

![示例][15]

第 100 个，感觉不错...

![示例][16]

500 个，嗯？CPU 好吃力啊。帧数也差不多 300 个左右的时候就降下来了。因为动画结束了就把 will-change 给删了，GPU 负荷是轻了，但是 CPU 负担却重了。

![示例][17]

到最后，差不多要“罢工”了。

![示例][18]

所有的动画结束了之后，终于回复到 60fps 了。

### 到底发生了什么？

好奇怪，结果并没有那么明朗。
我只是在动画结束后马上把 will-change 的值删了而已，到底发生了什么？

![示例][19]

看了下 Chrome 的 performance 的情况，Update Layer Tree 很迷，隔一段时间执行一次。

这是 Chrome 内部的处理，我也不太知道具体是为什么...

[看 DevTools 的 Timeline 面板、理解浏览器的渲染机制(日文链接)][20]

> Update Layer Tree <br>
> GPU 更新着的执行处理的 layer

就是说，GPU 为了执行处理，把需要处理的元素放到 layer 上，把不需要处理的元素从 layer 删掉，然后重新构建。 

这次试错的结果，现在的这个情况，Chrome 会有以下会倾向的点：

  - 依赖 layer 的元素，会使 Update Layer Tree 变重。
  - 比起升级到 layer，从 layer 降级更加耗费性能，也就是说删除 will-change 的工作很费性能。

我没有看源码，这些也只是我的猜测。

### 实验 3 结论

  - 动画结束了之后把 will-change 属性删掉，会影响 GPU 的负荷
  - GPU 负荷高的时候，再设置 will-change 的值会很吃力，特别是把 will-change 的指删掉
  - 高负荷状态下，动画结束了，一个一个的去删 will-change 的值，无疑是致命的

## 实验 4：某种程度下，删除 will-change 的值

在实验 3 中，动画结束，一个一个的去删 will-change 的值，效果会非常的糟糕。不知道为什么 Chrome 要在这个点上这么的“努力”啊，我试试看有没有其它办法绕一下。

既然执一次一次去删 will-change 的话，会让 Update Layer Tree 产生负担，那么 100 个的话也是一样的吧。那么在攒一定数量之后一口气的去删掉的话会是怎么样呢？马上来试试看。

准备好 Flower 队列

`Flower.vue`
``` javascript
const queueLimit = 100
const stopedFlowers: Flower[] = []
```

动画结束后，加到队列里， 到 100 个了之后，设置 isMoving 的值，一口气删掉 will-change 的值。

`Flower.vue`
``` javascript
private onEndAnim() {
  stopedFlowers.push(this)
  if (stopedFlowers.length === queueLimit) {
    stopedFlowers.forEach(fl => fl.isMoving = false)
    stopedFlowers.length = 0
  }
}
```

感觉有点像是在投机取巧，哈哈。实验开始！

![示例][21]

第 100 个，这个时候动画才开始，will-change 全部都是带着的，也就是说，现在的这个状态跟实验 2 是一样的。

![示例][22]

第 500 个，以 100 个为单位，删除 will-change，所以会隔一段时间，帧数会掉一下。

![示例][23]

到最后这种情况会一直持续，虽然会有那么一瞬间帧数会卡一下，单总的来说，负荷还是很平衡的。

![示例][24]

安全跑完，最后 100 个结束了之后，所有的元素都会删掉 will-change 的值。这时候，跟实验 1 和实验 3 的状态是一样的。

### 实验 4 结论

  - 如果 will-change 在动画结束后，攒到一定程度，再去删除的话，效果会比较好
  - 删掉 will-change 属性的那一瞬间，负荷会变重是没办法避免的（限本次实验范围）。
  - 在大幅度去使用动画的时候，看准时间去删掉 will-change，我认为是有必要的

## 最后顺便测一下 safari 下的情况吧

在 MacOS/iOS 的 Safari 跑了下，实验 3 的结果非常的流畅。

![示例][25]

也就是说，如果不是 Chrome 的 bug 的话，难道是我使用的问题？因为没有看源码，所以也不太确定。如果有知道的同学，欢迎在评论区评论。

## 总结

  - 想流畅的使用 GPU 做 CSS 动画的话，加上 will-change 属性吧。
  - will-change 在动画结束的时候，删掉这个属性吧。
  - 在 Chrome 下，will-change 删掉的话，Update Layer Tree 会重新构建 layer，负担会变重。可以每隔一段时间去删掉 will-change，可以把影响降到最低。
  - 多确认下不同的浏览器和不同的环境。不要总是想着“Chrome 即是正义”。
  - 夺取确认性能监测面板，还有系统的性能监测面板。就算有 60fps，说不定你的机器正在“惨叫”...


## 译者记

之前只是知道 will-change 会让浏览器启用 gpu 渲染，没想到这个使用多了，不一定就会爽了，使用 will-change 也是有很多需要注意的点。如果在需要大量使用动画的情况下，可以参考下这篇文章的做法，适当的去删掉 will-change 属性。

原文地址：https://qiita.com/yuneco/items/f07d60ab21dd1de9a51d


[1]:https://camo.qiitausercontent.com/cef8ef71d280c5804e358d95ea28a579d82db654/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f36306631643030312d656234362d623830612d376334652d3130316263343231663734392e676966
[2]:https://triple-underscore.github.io/css-will-change-ja.html
[3]:https://qiita.com/damele0n/items/71352757d0e6fdf5b184
[4]:https://www.webprofessional.jp/achieve-60-fps-mobile-animations-with-css3/
[5]:https://developers.google.com/web/fundamentals/performance/rendering/stick-to-compositor-only-properties-and-manage-layer-count?hl=ja
[6]:https://camo.qiitausercontent.com/f1b6d6aa347429a5c3c4542828a2a6f820c1cde2/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f64393832366363392d663437662d366238632d303965322d6665333430346635653038612e706e67
[7]:https://camo.qiitausercontent.com/e501af366a3f9503b7b97ae76a5c83e31bcd58cd/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f64303263323537652d663037362d386230352d653965622d3638303232636136643534322e6a706567
[8]:https://camo.qiitausercontent.com/b32c631076f8b7c2974437cadc48ac669fc7e1f2/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f32653364346330352d326338642d306638622d623334322d3263666665313164306233612e6a706567
[9]:https://camo.qiitausercontent.com/977fa25b2ae1ca9e83433471b8fdf657e6eac235/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f32313630323266612d363931662d343861382d316133642d3732343931633832323931652e6a706567
[10]:https://camo.qiitausercontent.com/4b9c4a60090770e6b7cd4e8ac20d9ba8a5c0a435/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f31316264386536662d393436652d306266312d333562352d6366643335313563656465372e6a706567
[11]:https://camo.qiitausercontent.com/6facbf2b79b65af306389acc8fdbb247cb10d758/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f61326332393962332d366331302d333761642d396637622d3534353737346135643365312e6a706567
[12]:https://camo.qiitausercontent.com/3c4445f1b4c38d392d9de1ddb95bea3c49f0d31e/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f33666430396430372d393433652d366534662d396432382d3162663266393936316436642e6a706567
[13]:https://camo.qiitausercontent.com/a8ac8d99800ee17b8ad252f0e7c887e3a0d45140/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f37666663633038392d656262302d666665322d393061622d3434363666666363326464662e6a706567
[14]:https://camo.qiitausercontent.com/b77e71e8a23778f1cfb31c4634733a1cbd62282a/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f66316536313230622d613761622d633865332d366164342d6637613365326439656430382e6a706567
[15]:https://camo.qiitausercontent.com/3726f7a5b14117613b98ea470162ba5d90017902/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f35303064323064352d323039632d666531662d393835382d3564363830303530383934392e6a706567
[16]:https://camo.qiitausercontent.com/2634d660e19225cd3e08d816b2ab473b819c3774/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f31343037346564382d633366362d303436632d653862322d3035393837656363393234302e6a706567
[17]:https://camo.qiitausercontent.com/ae03eef0e9cf90f54833c4b108d4b2baa19861a7/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f66383630333661632d643531332d643832652d383965322d3037343032353531616162352e6a706567
[18]:https://camo.qiitausercontent.com/6ad4bf7bd586ff216589447338b518c874d35b6b/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f35663632663338312d323630632d323964662d643533352d6137303830653136306362392e6a706567
[19]:https://camo.qiitausercontent.com/117bfe04eb9c4355d5aa0aaa3d51a42961d2220d/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f64386630373932342d336134302d643335302d323231302d3565366335663634376262632e6a706567
[20]:https://qiita.com/cy-mitsuki/items/51a0a4c17b89154a7af2
[21]:https://camo.qiitausercontent.com/d2a3e99757d09779d97f89b04c010b7ad397731f/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f37376633396361352d396265632d666631642d623330642d6164663962346661396230632e6a706567
[22]:https://camo.qiitausercontent.com/93028680fbf783852f4d4c37ff02c832794aa69c/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f37643436393235612d303131362d396433612d646635382d3262326135626462636432632e6a706567
[23]:https://camo.qiitausercontent.com/2a78718d513954195c7f50879f1ff70ff994d460/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f39663339313035372d633230662d613564642d633865332d3637323061613532363161362e6a706567
[24]:https://camo.qiitausercontent.com/ea117822075c59bf964b45069947dbded3573f6c/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f62383234383239372d393539382d613438342d613336312d6630666563643766346436342e6a706567
[25]:https://camo.qiitausercontent.com/ab063cddc126b369dc05ee73288d077efa0e181c/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e61702d6e6f727468656173742d312e616d617a6f6e6177732e636f6d2f302f3234353734302f38393039633535642d306233642d336533322d313134322d6535643831373938333336342e6a706567