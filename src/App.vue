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
      layout: [],
      rowNum: 0,
      colNum: 0,
      shape: [],
      shapeIndex: {
        x: 0,
        y: 0
      }
    }
  },
  created() {
    this.initLayout(10, 10)
    this.randomShape()
    this.initBlock()
    this.keyEvent()
    this.handleAnimation()
  },
  watch: {
    shapeIndex(val, oldVal) {

    }
  },
  methods: {
    initLayout(row, col) {
      let arr = []

      this.rowNum = row
      this.colNum = col

      for(let r = 0; r < row; r++) {
        arr.push([])
        for(let c = 0; c < col; c++) {
          arr[r].push(0)
        }
      }

      this.layout = arr
    },
    initBlock() {
      let mid = Math.ceil(this.rowNum / 2)
      let blockArr = [[0, 0, 1],[1, 1, 1]]
      let blockLen = blockArr[0].length
      let blockMid = Math.ceil(blockLen / 2)

      this.shapeIndex = {
        x: mid - 2,
        y: 0
      }

      for(let r = 0; r < this.shape.length; r++) {
        for(let c = 0; c < this.shape[r].length; c++) {
          if (this.shape[r][c] == 1) {
            this.layout[this.shapeIndex.y + r][this.shapeIndex.x + c] = this.shape[r][c]
          }
        }
      }
    },
    randomShape() {
      let random = Math.floor(Math.random() * (6 - 1) + 1)

      switch(random) {
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
    rotateShape() {
      let arr = []
      let arrLayout = this.layout


      for(let r = 0; r < this.shape.length; r++) {
        for(let c = 0; c < this.shape.length; c++) {
          if (this.shape[r][c] == 1) {
            arrLayout[this.shapeIndex.y + r][this.shapeIndex.x + c] = 0
          }
        }
      }

      this.layout.splice()

      for(let r = 0; r < this.shape.length; r++) {
        arr[r] = []
        for(let c = 0; c < this.shape[r].length; c++) {
          arr[r][c] = this.shape[this.shape.length - c - 1][r]
        }
      }

      this.shape = arr
      this.initBlock()

    },
    keyEvent() {
      document.addEventListener('keydown', e => {
        console.log(e);
        if (e.keyCode == 38) {
          this.rotateShape()
        }
      })
    },
    handleAnimation() {
      let animation = setInterval(() => {
        let arrLayout = this.layout
        for(let r = 0; r < this.shape.length; r++) {
          for(let c = 0; c < this.shape.length; c++) {
            if (this.shape[r][c] == 1) {
              arrLayout[this.shapeIndex.y + r][this.shapeIndex.x + c] = 0
            }
          }
        }

        let obj = {
          x: this.shapeIndex.x,
          y: this.shapeIndex.y
        }

        obj.y += 1

        for(let r = 0; r < this.shape.length; r++) {
          for(let c = 0; c < this.shape[r].length; c++) {
            if (this.shape[r][c] == 1) {
              this.layout[this.shapeIndex.y + r][this.shapeIndex.x + c] = this.shape[r][c]
            }
          }
        }
        this.layout.splice()
      }, 1000);

      return animation
    },
    collideDetection() {
      
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
</style>
