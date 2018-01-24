# vue-tetris

> 使用Vue实现的俄罗斯方块游戏

## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build
```

For detailed explanation on how things work, consult the [docs for vue-loader](http://vuejs.github.io/vue-loader).

## 在线访问

> [在线访问](https://lewis-geek.github.io/vue-tetris/)

![游戏界面](http://olnzpdi2u.bkt.clouddn.com/IMG_0662.png)


## 项目代码

[vue-tetris](https://github.com/lewis-geek/vue-tetris)

## 游戏简介

实现俄罗斯方块基本规则外，游戏增加了4种不同的色彩区别方块、键盘以及屏幕操作、游戏音效、积分系统、[实时通信引擎](https://docs.wilddog.com/sync/Web/index.html)实现的玩家排行榜等。同时支持PC和移动端。

## 操作介绍

### 键盘操作

Z键:旋转  
空格:暂停/显示排行榜  
左箭头:向左移动  
右箭头:向右移动  
上箭头:下坠  
下箭头:向下移动

### 屏幕操作

大按钮:旋转  
PAUSE:暂停/显示排行榜  
左按钮:向左移动  
右按钮:向右移动  
上按钮:下坠  
下按钮:向下移动

## 项目结构  

项目使用vue框架，使用到了vue的页面渲染、计算属性、watcher等功能。

```
.
├── README.md
├── index.html          # 页面入口
├── package.json        # npm相关
├── src     
│   ├── App.vue         # 主文件
│   ├── assets          # 静态资源
│   │   ├── clear.mov
│   │   ├── down.mov
│   │   └── over.mov
│   └── main.js
└── webpack.config.js   # webpack配置
```

## 实现思路

### 视图部分

游戏区对应一个二维数组，可约定两个量表示当前坐标是否有方块，例如1表示有方块，0表示没有方块。为了丰富游戏体验，也可将两个量改为多个量，区别不同的方块模型，为方块模型设置不同颜色，提高游戏趣味性。   

```javascript
let arrColor = []
for (let r = 0; r < row; r++) {
      arrColor.push([])
      for (let c = 0; c < col; c++) {
      arrColor[r].push('')
      }
}
```

### 方块模型

方块模型对应下图的几种形态，每个模型最多由4个小方块组成，将方块模型保存在4x4的二维数组中，使用6个不同的数组即可表示全部模型初始状态。  

我在组件中声明了一个shape属性，每次生成新的方块模型时，随机选择一个方块模型的二维数组，并赋值给shape。

```javascript
switch (reandom) {
      case 1:
      this.shape = [[0, 0, 0, 0], [0, 0, 1, 0], [1, 1, 1, 0], [0, 0, 0, 0]];
      break;
      case 2:
      this.shape = [[0, 0, 0, 0], [0, 1, 1, 0], [0, 1, 1, 0], [0, 0, 0, 0]];
      break;
      case 3:
      this.shape = [[0, 0, 0, 0], [0, 1, 0, 0], [0, 1, 1, 1], [0, 0, 0, 0]];
      break;
      case 4:
      this.shape = [[0, 0, 0, 0], [1, 1, 1, 1], [0, 0, 0, 0], [0, 0, 0, 0]];
      break;
      case 5:
      this.shape = [[0, 0, 0, 0], [0, 1, 1, 0], [0, 0, 1, 1], [0, 0, 0, 0]];
      break;
      case 6:
      this.shape = [[0, 0, 0, 0], [0, 0, 1, 0], [0, 1, 1, 1], [0, 0, 0, 0]];
      break;
}
```

![四格骨骼](http://olnzpdi2u.bkt.clouddn.com/3908048923-57dc200ce044c_articlex.png)


### 旋转算法

通过旋转算法，配合6个初始的方块模型，即可表示全部的方块图案。将一个4x4的矩阵旋转90度，推导对应坐标的转换关系即可。

```javascript
let rotatedShape = []   //保存旋转后的方块模型坐标
let shape = this.shape  //取当前的方块模型坐标

//循环遍历原方块模型二维数组，将旋转后的坐标保存在rotatedShape数组
for (let r = 0; r < shape.length; r++) {
      rotatedShape[r] = []
      for (let c = 0; c < shape[r].length; c++) {
      //坐标替换关系
      rotatedShape[r][c] = shape[shape.length - c - 1][r]  
      }
}
```

### 碰撞算法

当方块模型移动时进行碰撞检测，如果发生碰撞，则不能移动，需要碰撞检测算法。我的实现思路是，遍历移动后的方块模型坐标，在视图中检查对应坐标是否已有方块，同时要考虑边界，不能超出视图边界。

```javascript
function colDetection(shapeCoord) {
  let pass = true

  for (let coord of shapeCoord) {
    if (coord.y > this.rowNum - 1 || coord.y < 0) {
      return false
    }
    if (coord.x > this.colNum - 1 || coord.x < 0) {
      return false
    }
    if (this.layout[coord.y][coord.x] == 1) {
      pass = false
      for (let c of this.shapeCoord) {
        if (coord.y == c.y && coord.x == c.x) {
          pass = true
        }
      }
      if (pass == false) {
        return false
      }
    }
  }
  return pass
}
```   

### 移动和变形

定义一个坐标点为表示方块模型左上角顶点坐标，使用vue框架的计算属性，当方块模型或者顶点坐标发生变化时，自动计算新的视图。  
方块模型改变时，需要两步操作，先清除当前在坐标中的坐标，再将新的坐标添加到视图中。

## 总结

以上只是实现俄罗斯方块游戏的几个关键点，要完美的实现俄罗斯方块游戏还需要很多相对复杂的逻辑代码，不具体介绍了。
