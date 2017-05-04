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
      lastShape: []   //更新前的图形
    }
  },
  created() {
    this.initLayout(20, 10)
  },
  watch: {
    shape(val) {
      // console.table(val);
    },
    shapeCoord(val, oldVal) {
      for(let coord of oldVal) {
        this.layout[coord.y][coord.x] = 0
      }
      this.renderShape()
    }
  },
  computed: {
    shapeCoord() {
      let arr = this.computedCoord(this.shape, this.shapeIndex)

      // for(let r = 0; r < this.shape.length; r++) {
      //   for(let c = 0; c < this.shape[r].length; c++) {
      //     if (this.shape[r][c] == 1) {
      //       let obj = {
      //         x: this.shapeIndex.x + c,
      //         y: this.shapeIndex.y + r
      //       }
      //       arr.push(obj)
      //     }
      //   }
      // }

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
        y: 0
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
    randomShape() {
      let reandom = Math.floor(Math.random() * (6 - 1) + 1)

      switch(1) {
        case 1:
          this.shape = [[0,1,0,0],[0,1,0,0],[0,1,1,0],[0,0,0,0]];
          break;
        case 2:
          this.shape = [[0,0,0,0],[0,1,1,0],[0,1,1,0],[0,0,0,0]];
          break;
        case 3:
          this.shape = [[0,0,1,0],[0,0,1,0],[0,1,1,0],[0,0,0,0]];
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
        
      }
      function arrowRight() {
        console.log('right');
      }
      function arrowDown() {
        console.log('down');
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

      shapeCoord = this.computedCoord(shape, this.shapeIndex)
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
    colDetection(shapeCoord) {
      let pass = true

      for(let coord of shapeCoord) {
        if (coord.y == this.rowNum - 1) {
          pass = false
        }
        if (coord.x == this.colNum - 1 || coord.x == 0) {
          pass = false
        }
        if (this.layout[coord.y][coord.x] == 1) {
          pass = false
          for(let c of this.shapeCoord) {
            if (coord.y == c.y && coord.x == c.x) {
              pass = true
            }
          }
        }
      }
      return pass
    }
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
