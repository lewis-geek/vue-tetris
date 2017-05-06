<template>
  <div id="app">
    <div class="layout">
      <div
        class="layout-row"
        v-for="(row, rowIndex) in layout">
        <div
          class="layout-cell"
          :class="layoutColor[rowIndex][cellIndex]"
          v-for="(cell, cellIndex) in row">
      </div>
      </div>
    </div>
    <div class="control">
      <div class="control-left">
        <div class="button rotate-button">

        </div>
      </div>
      <div class="control-right">
        <div class="drop">
          <div class="button drop-button">

          </div>
        </div>
        <div class="left-right">
          <div class="button left-button">

          </div>
          <div class="button right-button">

          </div>
        </div>
        <div class="down">
          <div class="button down-button">

          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'app',
  data () {
    return {
      layout: [],     //布局
      rowNum: 0,
      colNum: 0,
      shape: [],      //当前图形
      shapeIndex: {
        x: 0,
        y: 0
      },              //图形顶点
      // shapeCoord: [], //图形的全部坐标
      lastShape: [],   //更新前的图形
      clear: true,
      intervalObj: -1,
      currentColor: '',
      layoutColor: [],
      lastReandomColor: 0,
      currentScore: 0,
      bestScore: 0,
      globalScore: [],
      clearLine: 0
    }
  },
  created() {
    this.initLayout(20, 10)
    // this.handleAnimation(1000)
  },
  mounted() {
    this.buttonEvent()
  },
  watch: {
    shape(val) {
      // console.table(val);
    },
    layoutColor(val) {
      // console.table(val)
    },
    shapeCoord(val, oldVal) {
      if (this.clear) {
        for(let coord of oldVal) {
          this.layout[coord.y][coord.x] = 0
          this.layoutColor[coord.y][coord.x] = ''
        }
      } else {
        let gameOver = false
        for (let coord of val) {
          if (this.layout[coord.y][coord.x] == 1) {
            gameOver = true
          }
        }

        if (gameOver) {
          clearInterval(this.intervalObj)
          alert('游戏结束');
          return
        } else {
          for(let r = 0; r < this.layout.length; r ++) {
            let clear = true
            for(let c = 0; c < this.layout[r].length; c++) {
               if (this.layout[r][c] == 0) {
                 clear = false
               }
            }
            if (clear) {
              let arr = []
              let arrColor = []
              this.layout.splice(r, 1)
              this.layoutColor.splice(r, 1)
              for(let i = 0; i < this.colNum; i++) {
                arr.push(0)
                arrColor.push('')
              }
              this.layout.unshift(arr)
              this.layoutColor.unshift(arrColor)
            }
          }
        }
      }

      this.clear = true
      this.renderShape()
    },
    currentScore(val) {
      if (val > this.bestScore) {
        this.bestScore = val
      }
    }
  },
  computed: {
    shapeCoord() {
      let arr = this.computedCoord(this.shape, this.shapeIndex)
      return arr
    }
  },
  methods: {
    /**
     * [初始化画布]
     * @param  {[Number]} row [行数]
     * @param  {[Number]} col [列数]
     */
    initLayout(row, col) {
      let arr = []
      let arrColor = []
      let mid = Math.ceil(col / 2)

      let shapeIndex = {
        x: mid - 2,
        y: -1
      }

      for(let r = 0; r < row; r++) {
        arr.push([])
        arrColor.push([])
        for(let c = 0; c < col; c++) {
            arr[r].push(0)
            arrColor[r].push('')
        }
      }

      this.rowNum = row
      this.colNum = col
      this.layout = arr
      this.layoutColor = arrColor
      this.shapeIndex = shapeIndex

      this.randomShape()
      this.keyEvent()
      // console.log(`顶点坐标 x:${shapeIndex.x} y:${shapeIndex.y}`);
    },
    initShapeIndex() {
      let mid = Math.ceil(this.colNum / 2)

      let shapeIndex = {
        x: mid - 2,
        y: -1
      }

      this.shapeIndex = shapeIndex
    },
    randomShape() {
      let reandom = Math.floor(Math.random() * (7 - 1) + 1)
      let reandomColor = reandomColor = Math.floor(Math.random() * (5 - 1) + 1)
      let lastReandomColor = this.lastReandomColor

      while (reandomColor == lastReandomColor) {
        reandomColor = reandomColor = Math.floor(Math.random() * (4 - 1) + 1)
      }

      switch (reandomColor) {
        case 1:
          this.currentColor = 'turquoise'
          break;
        case 2:
          this.currentColor = 'emerald'
          break;
        case 3:
          this.currentColor = 'peter-river'
          break;
        case 4:
          this.currentColor = 'amethyst'
          break;
      }

      this.lastReandomColor = reandomColor

      switch(reandom) {
        case 1:
          this.shape = [[0,0,0,0],[0,0,1,0],[1,1,1,0],[0,0,0,0]];
          break;
        case 2:
          this.shape = [[0,0,0,0],[0,1,1,0],[0,1,1,0],[0,0,0,0]];
          break;
        case 3:
          this.shape = [[0,0,0,0],[0,1,0,0],[0,1,1,1],[0,0,0,0]];
          break;
        case 4:
          this.shape = [[0,0,0,0],[1,1,1,1],[0,0,0,0],[0,0,0,0]];
          break;
        case 5:
          this.shape = [[0,0,0,0],[0,1,1,0],[0,0,1,1],[0,0,0,0]];
          break;
        case 6:
          this.shape = [[0,0,0,0],[0,0,1,0],[0,1,1,1],[0,0,0,0]];
          break;
      }

    },
    renderShape() {
      for(let coord of this.shapeCoord) {
        this.layout[coord.y][coord.x] = 1
        this.layoutColor[coord.y][coord.x] = this.currentColor
      }
      this.layoutColor.splice()
      this.layout.splice()
    },
    keyEvent() {
      document.addEventListener('keydown', e => {
        if (e.keyCode == 38) {
          arrowUp()
        }
        if (e.keyCode == 37) {
          arrowLeft()
        }
        if (e.keyCode == 39) {
          arrowRight()
        }
        if (e.keyCode == 40) {
          arrowDown()
        }
        if (e.keyCode == 32) {
          arrowPause()
        }
      })

      let that = this

      function arrowUp() {
        that.rotateShape()
      }
      function arrowLeft() {
        that.moveLeft()
      }
      function arrowRight() {
        that.moveRight()
      }
      function arrowDown() {
        let addScore = that.moveDown()
        if (addScore) {
          that.currentScore += 1
        }
      }
      function arrowPause() {
        console.log('pause');
      }
    },
    buttonEvent() {
      let rotateButton = document.querySelector('.rotate-button')
      let dropButton = document.querySelector('.drop-button')
      let leftButton = document.querySelector('.left-button')
      let rightButton = document.querySelector('.right-button')
      let downButton = document.querySelector('.down-button')

      rotateButton.addEventListener('click', () => {
        this.rotateShape()
      })

      leftButton.addEventListener('click', () => {
        this.moveLeft()
      })

      rightButton.addEventListener('click', () => {
        this.moveRight()
      })

      downButton.addEventListener('click', () => {
        let addScore = this.moveDown()
        if (addScore) {
          this.currentScore += 1
        }
      })
    },
    rotateShape() {
      let rotatedShape = []
      let shape = this.shape
      let shapeCoord = []
      let pass = true

      for(let r = 0; r < shape.length; r++) {
        rotatedShape[r] = []
        for(let c = 0; c < shape[r].length; c++) {
          rotatedShape[r][c] = shape[shape.length - c - 1][r]
        }
      }

      shapeCoord = this.computedCoord(rotatedShape, this.shapeIndex)
      pass = this.colDetection(shapeCoord)

      if (pass) {
        this.shape = rotatedShape
      }
    },
    computedCoord(shape, shapeIndex) {
      let arr = []

      for(let r = 0; r < shape.length; r++) {
        for(let c = 0; c < shape[r].length; c++) {
          if (shape[r][c] == 1) {
            let obj = {
              x: shapeIndex.x + c,
              y: shapeIndex.y + r
            }
            arr.push(obj)
          }
        }
      }
      return arr
    },
    /**
     * [碰撞检测的方法]
     * @param  {[Array]} shapeCoord [改变后的图形坐标]
     */
    moveDown() {
      let shapeCoord = []
      let shape = this.shape
      let pass = false
      let newShapeIndex = {
        x: this.shapeIndex.x,
        y: this.shapeIndex.y + 1
      }

      for (let coord of this.shapeCoord) {
        if (coord.y + 1 > this.rowNum - 1) {
          this.clear = false
          this.randomShape()
          this.initShapeIndex()
          return
        }
      }

      shapeCoord = this.computedCoord(this.shape, newShapeIndex)
      pass = this.colDetection(shapeCoord)

      if (pass) {
        this.shapeIndex = newShapeIndex
      } else {
        this.clear = false
        this.randomShape()
        this.initShapeIndex()
      }

      return pass
    },
    moveLeft() {
      let shapeCoord = []
      let shape = this.shape
      let pass = false
      let canDown = this.canMoveDown()
      let newShapeIndex = {
        x: this.shapeIndex.x - 1,
        y: this.shapeIndex.y
      }

      for (let coord of this.shapeCoord) {
        if (coord.x - 1 < 0) {
          return
        }
      }

      shapeCoord = this.computedCoord(this.shape, newShapeIndex)
      pass = this.colDetection(shapeCoord)

      if (pass) {
        this.shapeIndex = newShapeIndex
      } else {
        if (canDown) {
          return
        } else {
          this.clear = false
          this.randomShape()
          this.initShapeIndex()
        }
      }
    },
    moveRight() {
      let shapeCoord = []
      let shape = this.shape
      let pass = false
      let canDown = this.canMoveDown()
      let newShapeIndex = {
        x: this.shapeIndex.x + 1,
        y: this.shapeIndex.y
      }

      for (let coord of this.shapeCoord) {
        if (coord.x + 1 > this.colNum - 1) {
          return
        }
      }

      shapeCoord = this.computedCoord(this.shape, newShapeIndex)
      pass = this.colDetection(shapeCoord)

      if (pass) {
        this.shapeIndex = newShapeIndex
      } else {
        if (canDown) {
          return
        } else {
          this.clear = false
          this.randomShape()
          this.initShapeIndex()
        }
      }
    },
    handleAnimation(time) {
      this.intervalObj = setInterval(() => {
        this.moveDown()
      }, time);
    },
    canMoveDown() {
      let shapeCoord = []
      let shape = this.shape
      let pass = false
      let newShapeIndex = {
        x: this.shapeIndex.x,
        y: this.shapeIndex.y + 1
      }

      for (let coord of this.shapeCoord) {
        if (coord.y + 1 > this.rowNum - 1) {
          return false
        }
      }

      shapeCoord = this.computedCoord(this.shape, newShapeIndex)
      pass = this.colDetection(shapeCoord)


      if (pass) {
        return true
      } else {
        return false
      }
    },
    colDetection(shapeCoord) {
      let pass = true

      for(let coord of shapeCoord) {
        if (coord.y > this.rowNum - 1 || coord.y < 0) {
          return false
        }
        if (coord.x > this.colNum - 1 || coord.x < 0) {
          return false
        }
        if (this.layout[coord.y][coord.x] == 1) {
          pass = false
          for(let c of this.shapeCoord) {
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
    },
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

#app {
  background-color: #223436;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  padding: 40px 0;
  box-sizing: border-box;
}

.layout {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.layout-row {
  display: flex;
}

.layout-cell {
  width: 20px;
  height: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #192122;
  margin: .5px;
  border-radius: 2px;
}

.layout-cell.turquoise {
  background-color: #f1c40f
}

.layout-cell.emerald {
  background-color: #2ecc71
}

.layout-cell.peter-river {
  background-color: #3498db
}

.layout-cell.amethyst {
  background-color: #9b59b6
}

.control {
  display: flex;
}

.control > div {
  width: 50%;
}

.control-left {
  display: flex;
  align-items: center;
  justify-content: flex-end;
}

.control-left {
}

.control-left .rotate-button {
  width: 80px;
  height: 80px;
  margin-right: 40px;
  border-radius: 50%;
}

.control-right {
  display: flex;
  flex-direction: column;
}

.control-right > div {
  display: flex;
  width: 120px;
}
.control-right .drop,
.control-right .down {
  justify-content: center;
}

.control-right .left-right {
  justify-content: space-between;
}

.control-right .button{
  width: 40px;
  height: 40px;
  border-radius: 50%;
}

.control .button {
  position: relative;
  overflow: hidden;
  background-color: #fff;
  background: linear-gradient(rgb(222, 222, 222), rgb(253, 253, 253));
  box-shadow: 0 3px 5px rgba(0,0,0,0.25),
  inset 0 1px 0 rgba(255,255,255,0.3),
  inset 0 -5px 5px rgba(100,100,100,0.1),
  inset 0 5px 5px rgba(255,255,255,0.3);
}

.control .button:active::after {
  position: absolute;
  content: '';
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, .3);
}
</style>
