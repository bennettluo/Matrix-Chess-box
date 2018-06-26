<template>
  <div id="wrapper">
    <main>
      <div class="cur-player">
        Player {{ curUser }}
      </div>
      <div class="matrix-container">
        <div 
          v-for="(row, index) in items" 
          :key="`row-${index}`"
          class="box-row" 
        >
          <div 
            v-for="(col, idx) in row" 
            :key="`col-${index}-${idx}`" 
            :class="`box ${predictedClass(index, idx)}`" 
            @click="event => handleBoxClick(event, index, idx)" 
          >
            {{ col }}
          </div>
        </div>
      </div>
      <div class="btn-groups">
        <button 
          class="btn btn-predict" 
          @click="event => handlePredict(event, curUser, items, false)"
        >
          Predict how to get win
        </button>
        <button class="btn btn-reset" @click="reset">
          Reset
        </button>
      </div>
    </main>
  </div>
</template>

<script>

const LENGTH = 3,
  DIMENSION = 2,
  EMPTY = '',
  PLAYER_X = 'X',
  PLAYER_O = 'O',
  VERTICAL = 'vertical',
  HORIZONTAL = 'horizontal',
  DIAGONAL = 'diagonal',
  BACKDIAG = 'backdiag',
  PREDICTED = 'predicted',
  name = 'LandingPage'

export default {
  name,
  data () {
    return {
      items: [],
      curUser: PLAYER_X,
      predictedItems: []
    }
  },
  created () {
    this.items = this.generateMatrix(LENGTH, DIMENSION)
  },
  methods: {
    handlePredict (event, currentUser, matrix, toWin) {
      matrix.forEach((row, ridx) => {
        row.forEach((col, cidx) => {
          // Filtering selected items
          if (col !== EMPTY) return

          // Looping four directions of vertical, horizontal, diagonal, backdiag
          const condition = this.loopDirections(
            matrix, currentUser, ridx, cidx, toWin
          )
          if (!condition) return
          this.predictedItems.push([ridx, cidx])
        })
      })
      console.log(this.predictedItems)
    },
    loopDirections (matrix, currentUser, ridx, cidx, toWin) {
      let _diagonal = false,
        _backdiag = false,
        _vertical = false,
        _horizontal = false

      // Checking the diagonal points
      if (ridx === cidx)
        _diagonal = this.matchOthers(
          matrix, DIAGONAL, currentUser, ridx, toWin
        )

      // Checking the backdiag points
      if (ridx === LENGTH - 1 - cidx)
        _backdiag = this.matchOthers(
          matrix, BACKDIAG, currentUser, ridx, toWin
        )

      // Commonly there are two directions for each one
      _vertical = this.matchOthers(
        matrix, VERTICAL, currentUser, cidx, toWin
      )
      _horizontal = this.matchOthers(
        matrix, HORIZONTAL, currentUser, ridx, toWin
      )

      return _diagonal || _backdiag || _vertical || _horizontal
    },
    matchOthers (matrix, type, curUser, idx, toWin) {
      let _count = 0
      for (let p = 0; p < LENGTH; p++) {
        const config = {
          vertical: matrix[p][idx],
          horizontal: matrix[idx][p],
          diagonal: matrix[p][p],
          backdiag: matrix[LENGTH - 1 - p][p]
        }

        // To calculate the total number of this element
        // Match LENGTH will get win
        if (config[type] === curUser) _count += 1
        else continue
      }
      return toWin ? (_count === LENGTH) : (_count === LENGTH - 1)
    },
    handleBoxClick (event, x, y) {
      // Updating the items
      const curRow = this.items[x].slice(0)
      if (curRow[y] !== EMPTY) return
      curRow[y] = this.curUser
      this.$set(this.items, x, curRow)
      this.predictedItems = []

      // Checking this point whether it can get win
      const isWinPoint = this.loopDirections(
        this.items, this.curUser, x, y, true
      )
      if (isWinPoint) {
        alert(`Player ${this.curUser} wins`)
        this.reset()
      } else this.curUser = this.swapUser(this.curUser)
    },
    reset () {
      this.items = this.generateMatrix(LENGTH, DIMENSION)
      this.curUser = PLAYER_X
      this.predictedItems = []
    },
    generateMatrix (num, dim) {
      if (dim === 0) return
      return new Array(num).fill(
        dim === 1 ? EMPTY : this.generateMatrix(num, --dim)
      )
    },
    swapUser (user) {
      return user === PLAYER_X ? PLAYER_O : PLAYER_X
    },
    predictedClass (ridx, cidx) {
      return this.predictedItems.filter(x => x[0] === ridx && x[1] === cidx)[0]
        ? PREDICTED
        : EMPTY
    }
  }
}
</script>

<style>
@import url("https://fonts.googleapis.com/css?family=Source+Sans+Pro");

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: "Source Sans Pro", sans-serif;
}

#wrapper {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  min-width: 768px;
  padding: 60px 80px;
  background: radial-gradient(
    ellipse at top left,
    rgba(255, 255, 255, 1) 40%,
    rgba(229, 229, 229, 0.9) 100%
  );
}

main {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}

main > div {
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
  align-items: center;
  flex: 1;
}

main > .cur-player {
  margin-bottom: 10px;
  font-size: 2rem;
}

main > .btn-groups {
  flex-flow: row nowrap;
}

.btn-groups > .btn {
  position: relative;
  top: 10px;
  padding: 6px 12px;
  margin: 0 6px;
  border: none;
  border-radius: 4px;
  color: rgba(255, 253, 233, 0.8);
  outline: none;
  -webkit-transition: all 0.3s;
  transition: all 0.3s;
}

.btn-groups > .btn:hover {
  cursor: pointer;
}

.btn-groups > .btn.btn-predict {
  background: #ca911e;
}

.btn-groups > .btn.btn-predict:hover {
  background: #775105;
}

.btn-groups > .btn.btn-reset {
  background: #c0c0c0;
}

.btn-groups > .btn.btn-reset:hover {
  background: #424242;
}

.box-row {
  display: flex;
  flex-flow: row wrap;
}

.box {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 200px;
  height: 200px;
  border: solid 1px grey;
  font-size: 30px;
  -webkit-transition: all 0.3s;
  transition: all 0.3s;
}

.box.predicted {
  background: lightcoral;
}

.box:hover {
  background: lightgrey;
  cursor: pointer;
}
</style>
