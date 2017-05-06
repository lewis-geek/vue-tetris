<template>
  <div id="app">
    <div class="layout">
      <div
        class="layout-row"
        v-for="row in layout">
        <div
          class="layout-cell"
          v-for="cell in row">
          {{ cell }}
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
      intervalObj: -1
    }
  },
  created() {
    this.initLayout(20, 10)
    this.handleAnimation(1000)
  },
  watch: {
    shape(val) {
      // console.table(val);
    },
    shapeCoord(val, oldVal) {

      if (this.clear) {
        for(let coord of oldVal) {
          this.layout[coord.y][coord.x] = 0
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

              this.layout.splice(r, 1)
              for(let i = 0; i < this.colNum; i++) {
                arr.push(0)
              }
              this.layout.unshift(arr)
            }
          }
        }
      }

      this.clear = true
      this.renderShape()
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
      let mid = Math.ceil(col / 2)

      let shapeIndex = {
        x: mid - 2,
        y: -1
      }

      for(let r = 0; r < row; r++) {
        arr.push([])
        for(let c = 0; c < col; c++) {
            arr[r].push(0)
        }
      }

      this.rowNum = row
      this.colNum = col
      this.layout = arr
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
      let reandom = Math.floor(Math.random() * (6 - 1) + 1)

      switch(6) {
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
          this.shape = [[0,0,0,0],[0,1,1,1],[0,0,1,0],[0,0,0,0]];
          break;
      }

    },
    renderShape() {
      for(let coord of this.shapeCoord) {
        this.layout[coord.y][coord.x] = 1
      }
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
        let shapeCoord = []
        let shape = that.shape
        let pass = false
        let canDown = that.canMoveDown()
        let newShapeIndex = {
          x: that.shapeIndex.x - 1,
          y: that.shapeIndex.y
        }

        for (let coord of that.shapeCoord) {
          if (coord.x - 1 < 0) {
            return
          }
        }

        shapeCoord = that.computedCoord(that.shape, newShapeIndex)
        pass = that.colDetection(shapeCoord)

        if (pass) {
          that.shapeIndex = newShapeIndex
        } else {
          if (canDown) {
            return
          } else {
            that.clear = false
            that.randomShape()
            that.initShapeIndex()
          }
        }
      }
      function arrowRight() {
        let shapeCoord = []
        let shape = that.shape
        let pass = false
        let canDown = that.canMoveDown()
        let newShapeIndex = {
          x: that.shapeIndex.x + 1,
          y: that.shapeIndex.y
        }

        for (let coord of that.shapeCoord) {
          if (coord.x + 1 > that.colNum - 1) {
            return
          }
        }

        shapeCoord = that.computedCoord(that.shape, newShapeIndex)
        pass = that.colDetection(shapeCoord)

        if (pass) {
          that.shapeIndex = newShapeIndex
        } else {
          if (canDown) {
            return
          } else {
            that.clear = false
            that.randomShape()
            that.initShapeIndex()
          }
        }
      }
      function arrowDown() {
        that.moveDown()
      }
      function arrowPause() {
        console.log('pause');
      }
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
        if (coord.y > this.rowNum - 1) {
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

.layout {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
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
}
</style>
