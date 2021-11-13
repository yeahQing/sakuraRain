# sakuraRain

- 增加了对樱花雨实现部分的描述
- 增加了typed脚本实现键入打字效果

## 樱花雨部分实现

樱花雨主要通过一个渲染器类 **(RENDERER)** 和一个樱花类 **(CHERRY_BLOSSOM)** 实现。

### 渲染器实现

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
    if (!this.cherries[i].render(this.context)) { // 如果渲染方法返回false，就删除当前樱花对象
       this.cherries.splice(i, 1); // 将位置i的樱花对象删除
    }
  }
  
  if (--this.addingInterval == 0) {
    this.addingInterval = this.maxAddingInterval; // 重置添加间隔为最大间隔
    this.cherries.push(new CHERRY_BLOSSOM(this, false)); // 实例化一个新的樱花对象加入数组
  }
  
}

```

### 樱花实现

1. 樱花对象实现

樱花类的构造函数需要两个参数，1. 渲染器对象 2. 是否随机

```javascript

var CHERRY_BLOSSOM = function (renderer, isRandom) {
  this.renderer = renderer;
  this.init(isRandom); // 调用init方法
};

```

2. 樱花类的属性成员和函数成员(类原型)

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
    this.x = this.getRandomValue(-this.renderer.width, this.renderer.width);
    this.y = isRandom ? this.getRandomValue(0, this.renderer.height) : this.renderer.height * 1.5;
    this.z = this.getRandomValue(0, this.FAR_LIMIT);
    this.vx = this.getRandomValue(-2, 2);
    this.vy = -2;
    this.theta = this.getRandomValue(0, Math.PI * 2);
    this.phi = this.getRandomValue(0, Math.PI * 2);
    this.psi = 0;
    this.dpsi = this.getRandomValue(Math.PI / 600, Math.PI / 300);
    this.opacity = 0; // 透明度
    this.endTheta = false;
    this.endPhi = false;
    this.rippleCount = 0; // 当前波纹数量
    
    var axis = this.getAxis(),
    theta = this.theta + Math.ceil(-(this.y + this.renderer.height * this.SURFACE_RATE) / this.vy) * Math.PI / 500;
    theta %= Math.PI * 2;
    
    // hsl和hsla函数类似与rgb和rgba，也是定义颜色的方法，不过是使用色相、饱和度、亮度、透明度来定义颜色
    // 色相取值[0-360], 饱和度[0-100%]，亮度[0-100%],透明度[0-1]
    // Y的偏移 theta <= pi/2(1.57) 或者 theta >= 3*pi/2（4.71）取值为-40, 在1.57到4.71之间取值40
    this.offsetY = 40 * ((theta <= Math.PI / 2 || theta >= Math.PI * 3 / 2) ? -1 : 1);
    // Y的阈值
    this.thresholdY = this.renderer.height / 2 + this.renderer.height * this.SURFACE_RATE * axis.rate;
    // 实体颜色
    this.entityColor = this.renderer.context.createRadialGradient(0, 40, 0, 0, 40, 80);
    this.entityColor.addColorStop(0, 'hsl(330, 70%, ' + 50 * (0.3 + axis.rate) + '%)');
    this.entityColor.addColorStop(0.05, 'hsl(330, 40%,' + 55 * (0.3 + axis.rate) + '%)');
    this.entityColor.addColorStop(1, 'hsl(330, 20%, ' + 70 * (0.3 + axis.rate) + '%)');
    // 阴影颜色
    this.shadowColor = this.renderer.context.createRadialGradient(0, 40, 0, 0, 40, 80);
    this.shadowColor.addColorStop(0, 'hsl(330, 40%, ' + 30 * (0.3 + axis.rate) + '%)');
    this.shadowColor.addColorStop(0.05, 'hsl(330, 40%,' + 30 * (0.3 + axis.rate) + '%)');
    this.shadowColor.addColorStop(1, 'hsl(330, 20%, ' + 40 * (0.3 + axis.rate) + '%)');
    
  },
  
  getRandomValue: function (min, max) {
    // Math.random()返回[0,1)之间的数
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
    context.bezierCurveTo(-60, 20, -10, -60, 0, -20); // 三次贝塞尔曲线
    context.bezierCurveTo(10, -60, 60, 20, 0, 40); // 三次贝塞尔曲线
    context.fill(); // 填充
  
    // 绘制花瓣的边缘
    for (var i = -4; i < 4; i++) {
      context.beginPath();
      context.moveTo(0, 40);
      context.quadraticCurveTo(i * 12, 10, i * 4, -24 + Math.abs(i) * 2); // 二次贝塞尔曲线
      context.stroke(); // 绘制边缘不填充
    }
    
  }，
  
  // 渲染樱花 由樱花对象在渲染器中调用
  render: function(context) {
    var axis = this.getAxis();
    
    // 如果y轴等于阈值Y，且波纹个数小于最大波纹个数, 绘制波纹
    if (axis.y == this.thresholdY && this.rippleCount < this.MAX_RIPPLE_COUNT) {
      context.save();
      context.lineWidth = 2;
      context.strokeStyle = 'hsla(0, 0%, 100%, ' + (this.MAX_RIPPLE_COUNT - this.rippleCount) / this.MAX_RIPPLE_COUNT + ')';
      context.translate(axis.x + this.offsetY * axis.rate * (this.theta <= Math.PI ? -1 : 1), axis.y); // 重新在轴x,y绘制
      context.scale(1, 0.3);
      context.beginPath();
      context.arc(0, 0, this.rippleCount / this.MAX_RIPPLE_COUNT * this.RIPPLE_RADIUS * axis.rate, 0, Math.PI * 2, false);
      context.stroke();
      context.restore();
      this.rippleCount++;
    }
    
    if (axis.y < this.thresholdY || (!this.endTheta || !this.endPhi)) {
      if (this.y <= 0) {
        this.opacity = Math.min(this.opacity + 0.01, 1);
      }
      context.save();
      context.globalAlpha = this.opacity;
      context.fillStyle = this.shadowColor;
      context.strokeStyle = 'hsl(330, 30%,' + 40 * (0.3 + axis.rate) + '%)';
      context.translate(axis.x, Math.max(axis.y, this.thresholdY + this.thresholdY - axis.y));
      context.rotate(Math.PI - this.theta);
      context.scale(axis.rate * -Math.sin(this.phi), axis.rate);
      context.translate(0, this.offsetY);
      this.renderCherry(context, axis); // 绘制樱花
      context.restore();
    }
    
    context.save(); // 保存当前状态
    context.fillStyle = this.entityColor;
    context.strokeStyle = 'hsl(330, 40%,' + 70 * (0.3 + axis.rate) + '%)';
    context.translate(axis.x, axis.y + Math.abs(this.SINK_OFFSET * Math.sin(this.psi) * axis.rate));
    context.rotate(this.theta);
    context.scale(axis.rate * Math.sin(this.phi), axis.rate);
    context.translate(0, this.offsetY);
    this.renderCherry(context, axis);  // 绘制樱花
    context.restore();  // 返回之前保存过的状态
    
    if (this.y <= -this.renderer.height / 4) {
        // 修改theta和endTheta的状态
        if (!this.endTheta) {
          for (var theta = Math.PI / 2, end = Math.PI * 3 / 2; theta <= end; theta += Math.PI) {
            if (this.theta < theta && this.theta + Math.PI / 200 > theta) {
              this.theta = theta;
              this.endTheta = true;
              break;
            }
          }
        }
        
        // 修改phi和endPhi的状态
        if (!this.endPhi) {
          for (var phi = Math.PI / 8, end = Math.PI * 7 / 8; phi <= end; phi += Math.PI * 3 / 4) {
            if (this.phi < phi && this.phi + Math.PI / 200 > phi) {
              this.phi = Math.PI / 8;
              this.endPhi = true;
              break;
            }
          }
        }
        
     } // endif
     
    // 如果endTheta为false
    if (!this.endTheta) {
    
      if (axis.y == this.thresholdY) {
        this.theta += Math.PI / 200 * ((this.theta < Math.PI / 2 || (this.theta >= Math.PI && this.theta < Math.PI * 3 / 2)) ? 1 : -1);
      } 
      else {
        this.theta += Math.PI / 500;
      }
      
      this.theta %= Math.PI * 2;
    }
    
    // 如果endPhi为true
    if (this.endPhi) {
      if (this.rippleCount == this.MAX_RIPPLE_COUNT) {
        this.psi += this.dpsi;
        this.psi %= Math.PI * 2;
      }
    }
    // 如果endPhi为false
    else {
      this.phi += Math.PI / ((axis.y == this.thresholdY) ? 200 : 500);
      this.phi %= Math.PI;
    }
    
    // 如果y小于画布高度乘以表面率
    if (this.y <= -this.renderer.height * this.SURFACE_RATE) {
      this.x += 2;
      this.y = -this.renderer.height * this.SURFACE_RATE;
    } 
    else {
      this.x += this.vx;
      this.y += this.vy;
    }
    
    return this.z > -this.FOCUS_POSITION && this.z < this.FAR_LIMIT && this.x < this.renderer.width * 1.5;
    
  }, // end render
  
}; // end CHERRY_BLOSSOM.prototype

```



