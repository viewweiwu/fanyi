# 【译】用 will-change 目标 60fps 如丝般顺化的 css 动画

大家好、这里就是用 css 和 Vue 做了疯狂的使用 css 动画的个人主页、还有射击游戏的那个 `yuki`。

感觉 css 的动画用的太多了，我的 MacBook 都要“罢工”了，反省过后呢，接下来我就给大家总结一下，我的实验结果吧。(还是好想用 css 动画

## 这次的例子

这篇文章的目的大概就是，在网页里用 css 动画加一点复杂的动画，然后去做一些游戏或者艺术表现的时候，全部都用上 css 的 will-change 属性，怎么样让 GPU 渲染最合适。

你要问 will-change 是什么？那可以看一下我放在这篇文章后面的参考列表。

![示例][1]

https://css-anime.firebaseapp.com/

本次要验证的动画有下面几个点：

  - 用 css 画 1000 朵花❀，一朵一朵的开
  - 每朵花都是 div 元素（这次不用 svg）
  - 每次间隔 32ms 画一朵
  - 开完之后的花，就保持停留在画面上
  - 并且画面一直会在旋转

不知道大家有没有遇到过，需要一边做动画，还得一直要加 html 标签，然后还得一直显示，且不能从画面消失，这种很痛苦的情况。

实验环境如下：

  - MacBookPro 2017Late / 8GB RAM（经常被叫做“梅”的最低配置）
  - MacOSX Mojave
  - Chrome 74

之后会提一下 Safari 跟 iOS 的情况。
因为不同的环境会有很大的差异，浏览器和操作系统不一样的话，结果会有差异，我先在这里事先说明一下。

## 大神们的智慧：我觉得一起读了也不错的东西

```!
译者注：下面链接全部都是*日文*的，标题我先翻译了一下，*日文* ok 的同学可以点点看...
```

这篇文章参考以下链接：

  - [CSS Will Change Module Level 1（日文版）][2]
  - [CSS: will-change 对性能的影响（日文版）][3]
  - [高性能 60fps 的 CSS3 动画最佳实践（日文版）][4]
  - [优先使用合成专用的属性，以及层级数的管理（日文版）][5]

## 实验0：基本元素与动画的构建

为了下手方便，我就先随便的写了一下，为了展示具体是什么在动，我把代码贴上来了，大家顺着代码看下就明白了。

花❀的话 html 大概就是这样的，为了轻松&个人兴趣，这里用了下 Vue，其实什么框架都无所谓。

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

花瓣是用 div 搭的，除了角度和颜色实在 template 里面写的，其它的都是在下面 \<style\> 标签里写的。开花的动画也在这里写了（也就是说，动画跟 Vue 和 javascirpt 没有关系，就只是用 css 写的）。

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

稍稍有些乏味，写出来的花大概就是这种感觉。

![示例][6]

让这个 Flower 组件每隔一定时间，就往画面里加，然后画面的整体让它转起来。

## 实验1：直接跑起来（没有写 will-change）

先试试不用 will-change，跑一下动画，Chrome 的 performance monitor 面板打开。

![示例][7]

```!
译者注：没有找到 gpu 面板，可以 f12 -> ctrl + shift + p 输入 rendering 打开 fps 面板，另外在 performance monitor 我只找到了 CPU 面板可以看，不知道图片的 gpu 面板怎么打开的。
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

教科书上也写着，但是不试试看怎么会知道呢？变更的地方就是在 style 里加上 will-change: transform。

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

第 100 个，非常都流畅，CPU 负荷也很低。

![示例][12]

500 个，开始有点挣扎了，GPU 就像吃了芥末一样，一下子就窜起来了，CPU 也有点负荷了。

![示例][13]

快到结尾了，CPU 和 GPU 都在疯狂的“惨叫”，performance 也是很危险的状态。

![示例][14]

所有的开花的动画都结束了之后，负荷也降不下来， 写了 will-change 的话，为了保证接下来的随时会开始的动画的性能，就算动画结束了，负荷也不会下降。







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