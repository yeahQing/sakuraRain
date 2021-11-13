# sakuraRain

- 增加了typed脚本实现键入打字效果

## 樱花雨部分实现

樱花雨主要通过一个渲染器类 **(RENDERER)** 和一个樱花类 **(CHERRY_BLOSSOM)** 实现。

1. 使用Jquery调用渲染器

```javascript

$(function () {
  RENDERER.init();
});

```

2. 渲染器类实现

``` javascript

var RENDERER = {
  // 数据成员
  INIT_CHERRY_BLOSSOM_COUNT: 30, // 初始化樱花的个数
  MAX_ADDING_INTERVAL: 10,
  
  // 函数成员
  
  // 初始化
  init: function () {
        this.setParameters();
        this.reconstructMethods();
        this.createCherries();
        this.render();
  },
  
  // 设置参数
  setParameters: function () {
        this.$container = $('#jsi-cherry-container');  // 根据id获取到div容器
        this.width = this.$container.width(); // 获取div容器的宽度
        this.height = this.$container.height(); // 获取div容器的高度
        this.context = $('<canvas />').attr({ width: this.width, height: this.height }).appendTo(this.$container).get(0).getContext('2d');  //  向div dom 中添加canvas画布
        this.cherries = [];  // 初始化存储樱花对象的数组
        this.maxAddingInterval = Math.round(this.MAX_ADDING_INTERVAL * 1000 / this.width); // 最大添加间隔
        this.addingInterval = this.maxAddingInterval; // 添加间隔
      },
  
};

```

3. 渲染器实例化(绘制)樱花对象

```javascript

createCherries: function () {
        for (var i = 0, length = Math.round(this.INIT_CHERRY_BLOSSOM_COUNT * this.width / 1000); i < length; i++) {
          this.cherries.push(new CHERRY_BLOSSOM(this, true));  // 将实例化的樱花对象加入数组
        }
}


```

4. 渲染器渲染函数实现

这里需要理解bind()方法，举个例子

假设现在有一个函数, 需要3个参数

```javascript

function foo(a, b, c) {
  return a + b + c;
}

```

现在我们使用bind方法创建一个新的函数，可以理解为函数指针_foo

```javascript

var _foo = foo.bind(this, 10);  // 第一个参数是当前对象的this指针, 之后的参数就是foo函数的参数列表

var ans = _foo(20, 30); // 这里绑定函数传递了10作为a的实参，20作为b的实参, 30作为c的实参

console.log(ans) // ans = 60

```

理解了bind函数，我们就可以将当前的render函数绑定当前RENDERER对象，生成一个新的函数

```javascript

reconstructMethods: function () {
  this.render = this.render.bind(this); // 将当前对象的render函数绑定当前RENDERER对象，生成一个新的函数
}

```

我们还需要理解requestAnimationFrame这个浏览器内置的函数，这个函数需要一个回调函数作为参数，在我们的例子中，
就是把render函数作为回调函数传入，因为我们每一帧都要渲染一遍樱花雨动画。

理解了以上这些，我们就可以实现render函数

``` javascript

render: function() {
  requestAnimationFrame(this.render); // 将render成员函数作为回调函数,每帧都调用一次
  this.context.clearRect(0, 0, this.width, this.height); // 清空画布
  
  this.cherries.sort(function (cherry1, cherry2) { // 按两个樱花对象的z轴排序
    return cherry1.z - cherry2.z;
  });
  
  for (var i = this.cherries.lenght - 1; i >= 0; i--) {
    if (!this.cherries[i].render(this.context)) {
       this.cherries.splice(i, 1); // 将位置i的樱花对象删除
    }
  }
  
  if (--this.addingInterval == 0) {
    this.addingInterval = this.maxAddingInterval; // 重置添加间隔为最大间隔
    this.cherries.push(new CHERRY_BLOSSOM(this, false)); // 实例化一个新的樱花对象加入数组
  }
  
}

```

5. 樱花对象实现

樱花类的构造函数需要两个参数，1. 渲染器对象 2. 是否随机

```javascript

var CHERRY_BLOSSOM = function (renderer, isRandom) {
  this.renderer = renderer;
  this.init(isRandom); // 调用init方法
};

```

樱花类的属性成员和函数成员(类原型)

```javascript

CHERRY_BLOSSOM.prototype = {
  
  // 属性成员
  FOCUS_POSITION: 300,
  FAR_LIMIT: 600,
  MAX_RIPPLE_COUNT: 100, // 最大波纹数量
  RIPPLE_RADIUS: 100, // 波纹半径
  SURFACE_RATE: 0.5,
  SINK_OFFSET: 20, // 下降偏移
  
  // 函数成员
  init: function (isRandom) {
    // 初始化方法会调用getRandomValue和getAxis获取随机值和轴线
  },
  
  getRandomValue: function (min, max) {
    return min + (max - min) * Math.random();
  },
  
  getAxis: function () {
    var rate = this.FOCUS_POSITION / (this.z + this.FOCUS_POSITION),
          x = this.renderer.width / 2 + this.x * rate,
          y = this.renderer.height / 2 - this.y * rate;
    return { rate: rate, x: x, y: y };
  },
  
  // 绘制樱花方法
  renderCherry: function (context) {
    // 使用canvas画布绘制樱花，利用贝塞尔曲线
    // 绘制花瓣的形状
    context.beginPath();
    context.moveTo(0, 40);
    context.bezierCurveTo(-60, 20, -10, -60, 0, -20);
    context.bezierCurveTo(10, -60, 60, 20, 0, 40);
    context.fill();
  
    // 绘制花瓣的边缘
    for (var i = -4; i < 4; i++) {
      context.beginPath();
      context.moveTo(0, 40);
      context.quadraticCurveTo(i * 12, 10, i * 4, -24 + Math.abs(i) * 2);
      context.stroke();
    }
    
  }，
  
  // 渲染樱花 由樱花对象在渲染器中调用
  render: function(context) {
    // TODO
  }
  
};

```



