<template>
  <div id="app">
    <div class="layout-container"
      v-show="!rankView">
      <div class="layout"
        :class="{'game-over': gameOver}">
        <div class="layout-row"
          v-for="(row, rowIndex) in layout">
          <div class="layout-cell"
            :class="layoutColor[rowIndex][cellIndex]"
            v-for="(cell, cellIndex) in row">
          </div>
        </div>
      </div>
      <div class="layout-right">
        <div class="best-score">
          <span>BEST</span>
          {{ this.bestScore }}
        </div>
        <div class="current-score">
          <span>SCORE</span>
          {{ this.currentScore }}
        </div>
        <div class="clear-line">
          <span>LINE</span>
          {{ this.clearLine }}
        </div>
        <div class="globale-score">
          <span>PAUSE</span>
          RANK
        </div>
      </div>
    </div>
    <div class="control"
      v-show="!rankView">
      <div class="control-left">
        <div class="button rotate-button"></div>
      </div>
      <div class="control-right">
        <div class="drop">
          <div class="button drop-button"></div>
        </div>
        <div class="left-right">
          <div class="button left-button"></div>
          <div class="button right-button"></div>
        </div>
        <div class="down">
          <div class="button down-button"></div>
        </div>
      </div>
    </div>
    <div class="rank-view"
      v-show="rankView">
      <div class="rank-container">
        <div class="person">
          <div class="score">
            Score
          </div>
          <div class="line">
            Line
          </div>
          <div class="string">
            Player
          </div>
        </div>
        <div class="person"
          v-for="item in this.globalScore">
          <div class="score">
            {{item.score}}
          </div>
          <div class="line">
            {{item.line}}
          </div>
          <div class="string">
            {{item.string}}
          </div>
        </div>
      </div>
      <div class="rank-view-close">
        X
      </div>
    </div>
    <audio class="audio-down"
      src="./dist/down.mov"></audio>
    <audio class="audio-clear"
      src="./dist/clear.mov"></audio>
    <audio class="audio-over"
      src="./dist/over.mov"></audio>
  </div>
</template>

<script>
import wilddog from 'wilddog'
import { MessageBox } from 'mint-ui'
import 'mint-ui/lib/style.css'
import './assets/down.mov'
import './assets/clear.mov'
import './assets/over.mov'

export default {
  name: 'app',
  data() {
    return {
      layout: [],     //布局
      rowNum: 0,
      colNum: 0,
      shape: [],      //当前图形
      shapeIndex: {
        x: 0,
        y: 0
      },              //图形顶点
      //shapeCoord: [], //图形的全部坐标
      lastShape: [],   //更新前的图形
      clear: true,
      intervalObj: -1,
      currentColor: '',
      layoutColor: [],
      lastReandomColor: 0,
      currentScore: 0,
      bestScore: 0,
      globalScore: [],
      clearLine: 0,
      $ref: '',
      rankView: false,
      audioDown: '',
      audioClear: '',
      audioOver: '',
      diffIndexY: 0,
      gameOver: false,
      initScore: false
    }
  },
  created() {
    let bestScore = localStorage.getItem('bestScore')
    if (!!bestScore) {
      this.bestScore = bestScore
    }
    this.initLayout(19, 10)
    this.handleAnimation(800)
    this.keyEvent()
    this.initData()
  },
  mounted() {

    this.audioDown = document.querySelector('.audio-down')
    this.audioClear = document.querySelector('.audio-clear')
    this.audioOver = document.querySelector('.audio-over')

    this.buttonEvent()
  },
  watch: {
    shapeCoord(val, oldVal) {
      if (this.clear) {
        for (let coord of oldVal) {
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
          this.handleGameOver()
          return
        } else {
          for (let r = 0; r < this.layout.length; r++) {
            let clear = true
            for (let c = 0; c < this.layout[r].length; c++) {
              if (this.layout[r][c] == 0) {
                clear = false
              }
            }
            if (clear) {
              let arr = []
              let arrColor = []
              this.audioClear.currentTime = 0
              this.audioClear.play()
              this.layout.splice(r, 1)
              this.layoutColor.splice(r, 1)
              for (let i = 0; i < this.colNum; i++) {
                arr.push(0)
                arrColor.push('')
              }
              this.clearLine += 1
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
        localStorage.setItem('bestScore', val)
        console.log('best:' + val);
      }
    },
    clearLine(val, oldVal) {
      let clearLineNum = val - oldVal
      if (!this.initScore) {
        this.currentScore += clearLineNum * 100 * clearLineNum
      } else {
        this.initScore = false
      }
    },
    globalScore(val) {
      console.log(JSON.stringify(val));
    },
    shapeIndex(val, oldVal) {
      this.diffIndexY = val.y - oldVal.y
    },
  },
  computed: {
    shapeCoord() {
      let arr = this.computedCoord(this.shape, this.shapeIndex)
      return arr
    },
  },
  methods: {
    /**
     * [初始化画布]
     * @param  {[Number]} row [行数]
     * @param  {[Number]} col [列数]
     */
    initData() {
      let config = {
        syncURL: 'https://vue-tetris.wilddogio.com'
      }
      wilddog.initializeApp(config)
      this.$ref = wilddog.sync().ref()

      this.$ref.on('value', (snapshot) => {
        let data = snapshot.val()
        this.globalScore = JSON.parse(data.globalScore);
      })
    },
    initLayout(row, col) {
      let arr = []
      let arrColor = []
      let mid = Math.ceil(col / 2)

      let shapeIndex = {
        x: mid - 2,
        y: -1
      }

      for (let r = 0; r < row; r++) {
        arr.push([])
        arrColor.push([])
        for (let c = 0; c < col; c++) {
          arr[r].push(0)
          arrColor[r].push('')
        }
      }

      this.rowNum = row
      this.colNum = col
      this.layout = arr
      this.layoutColor = arrColor
      this.shapeIndex = shapeIndex
      this.currentScore = 0
      this.clear = true
      this.clearLine = 0
      this.randomShape()
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

    },
    renderShape() {
      for (let coord of this.shapeCoord) {
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
        if (e.keyCode == 90) {
          keyZ()
        }
      })

      let that = this

      function arrowUp() {
        that.moveDrop()
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
        that.rankView = !that.rankView
        if (that.rankView) {
          clearInterval(that.intervalObj)
        } else {
          that.handleAnimation(800)
        }
      }
      function keyZ() {
        that.rotateShape()
      }
    },
    buttonEvent() {
      let rotateButton = document.querySelector('.rotate-button')
      let dropButton = document.querySelector('.drop-button')
      let leftButton = document.querySelector('.left-button')
      let rightButton = document.querySelector('.right-button')
      let downButton = document.querySelector('.down-button')
      let rankButton = document.querySelector('.globale-score')
      let rankClose = document.querySelector('.rank-view-close')
      let layout = document.querySelector('.layout')

      layout.addEventListener('click', () => {
        if (this.gameOver) {
          this.currentScore = 0
          this.initLayout(19, 10)
          this.handleAnimation(800)
          this.gameOver = false
        }
      })

      rotateButton.addEventListener('click', () => {
        this.rotateShape()
      })

      dropButton.addEventListener('click', () => {
        this.moveDrop()
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

      rankButton.addEventListener('click', () => {
        this.rankView = !this.rankView
        clearInterval(this.intervalObj)
      })

      rankClose.addEventListener('click', () => {
        this.rankView = !this.rankView
        this.handleAnimation(800)
      })
    },
    rotateShape() {
      let rotatedShape = []
      let shape = this.shape
      let shapeCoord = []
      let pass = true

      for (let r = 0; r < shape.length; r++) {
        rotatedShape[r] = []
        for (let c = 0; c < shape[r].length; c++) {
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

      for (let r = 0; r < shape.length; r++) {
        for (let c = 0; c < shape[r].length; c++) {
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
    moveDrop() {
      let pass = true

      let ID = setInterval(() => {
        let diffIndexY = this.diffIndexY
        pass = this.moveDown()
        this.currentScore += 5

        if (diffIndexY < 0) {
          clearInterval(ID)
        }

        for (let coord of this.shapeCoord) {
          if (coord.y + 1 > this.rowNum - 1) {
            clearInterval(ID)
          }
        }

        if (pass == false) {
          clearInterval(ID)
        }
      }, 0)
    },
    handleAnimation(time) {
      this.intervalObj = setInterval(() => {
        this.moveDown()
      }, time);
    },
    handleGameOver() {
      let lastGlobalScore = this.globalScore[this.globalScore.length - 1].score
      this.audioOver.currentTime = 0
      this.audioOver.play()
      this.initScore = true

      if (lastGlobalScore <= this.currentScore) {
        MessageBox.alert('恭喜您进入全球榜').then(action => {
          MessageBox.prompt('请留下您的大名').then((value, action) => {
            this.gameOver = true
            let string = value.value
            let obj = {
              score: this.currentScore,
              line: this.clearLine,
              string: string
            }

            this.globalScore.pop()
            this.globalScore.push(obj)

            this.globalScore.sort((a, b) => {
              return b.score - a.score
            })

            let globalScoreJSON = JSON.stringify(this.globalScore);

            this.$ref.set({
              'globalScore': globalScoreJSON
            })
          }).catch(e => {
            console.log(e)
            this.currentScore = 0
            this.gameOver = true
          })
        })
      } else {
        this.gameOver = true
      }
    },
    canMoveDown() {
      let shapeCoord = []
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
    },
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
}

body {
  font-family: "SF Pro SC", "SF Pro Text", "SF Pro Icons", "PingFang SC", "Helvetica Neue", "Helvetica", "Arial", sans-serif;
}

#app {
  background-color: #223436;
  height: 100vh;
  display: flex;
  flex-direction: column;
  padding: 15px 0 10px;
  box-sizing: border-box;
}

.layout-container {
  display: flex;
  justify-content: center;
}

.layout {
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border: 1px solid #000;
  border-radius: 3px;
  padding: 5px;
}

.layout.game-over::after {
  content: '再来一局';
  position: absolute;
  left: 50%;
  top: 50%;
  padding: 5px;
  border-radius: 5px;
  color: #fff;
  transform: translate(-50%, -140%);
}

.layout.game-over::before {
  content: '';
  position: absolute;
  right: 0;
  left: 0;
  bottom: 0;
  top: 0;
  background-color: rgba(0, 0, 0, .5)
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

.layout-right {
  margin-left: 10px;
}

.layout-right>div {
  display: flex;
  flex-direction: column;
  border: 1px solid #000;
  justify-content: center;
  align-items: center;
  margin-bottom: 5px;
  border-radius: 3px;
}

.control {
  display: flex;
  margin-top: 10px;
}

.control>div {
  width: 50%;
}

.control-left {
  display: flex;
  align-items: center;
  justify-content: flex-end;
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

.control-right>div {
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

.control-right .button {
  width: 45px;
  height: 45px;
  border-radius: 50%;
}

.control .button {
  position: relative;
  overflow: hidden;
  background-color: #fff;
  background: linear-gradient(rgb(222, 222, 222), rgb(253, 253, 253));
  box-shadow: 0 3px 5px rgba(0, 0, 0, 0.25),
  inset 0 1px 0 rgba(255, 255, 255, 0.3),
  inset 0 -5px 5px rgba(100, 100, 100, 0.1),
  inset 0 5px 5px rgba(255, 255, 255, 0.3);
  user-select: none;
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

.control .drop-button {
  transform: translateY(5px);
}

.control .down-button {
  transform: translateY(-5px);
}

.rank-view {
  margin-top: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #edd;
}

.rank-container {
  padding: 10px 5px;
  border: 1px solid #edd;
  border-radius: 3px;
}

.rank-view .person {
  display: flex;
  justify-content: space-between;
  align-items: center;
  min-width: 65vw;
  text-align: center;
  margin-bottom: 10px;
}

.rank-view .person>div {
  min-width: 4em;
}

.rank-view-close {
  margin-top: 20px;
  width: 25px;
  height: 25px;
  border: 1px solid #edd;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
