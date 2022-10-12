<template>
  <div>
    <grid-display ref="grid" :grid="displayGrid">
      <template #cell="{ value, x, y }">
        <grid-cell :value="value" @click="log(x, y, value)" />
      </template>
    </grid-display>
  </div>
</template>

<script>
import GridCell from './GridCell.vue'
import GridDisplay from './GridDisplay.vue'
export default {
  components: { GridDisplay, GridCell },
  data() {
    return {
      completeGrid: new Array(9).fill(0).map(() => new Array(9).fill(0)),
      playerGrid: new Array(9).fill(0).map(() => new Array(9).fill(0)),
      displayGrid: [],
      notesGrid: new Array(27).fill(0).map(() => new Array(27).fill(0)),
      subGrids: {},
      showWarnings: false,
      solutions: 0,
    }
  },
  created() {
    this.calcSubGrids()
    this.fillGrid()
    this.displayGrid = this.completeGrid.slice()
    this.removeCells(1)
  },

  methods: {
    log(x, y, value) {
      console.log(x, y, value)
    },
    calcSubGrids() {
      // calculates all the sub grids and stores position of the cells

      let row = 1 // start at 1 to be in middle of sub grid
      let col = 1

      let sub = 1
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          this.subGrids[sub] = []
          for (let x = -1; x < 2; x++) {
            for (let y = -1; y < 2; y++) {
              this.subGrids[sub].push([row + x, col + y])
            }
          }
          sub++ // move to next sub grid
          col += 3 // move to middle of next sub grid in the row
        }
        row += 3 // move to middle of next sub grid in column, starting a new row
        col = 1 // reset col to middle of first subgrid in new row
      }
    },
    checkGrid(grid) {
      for (let x = 0; x < grid.length; x++) {
        for (let y = 0; y < grid[x].length; y++) {
          if (grid[x][y] === 0) return false
        }
      }
      return true
    },
    fillGrid() {
      let row, col
      let numList = [1, 2, 3, 4, 5, 6, 7, 8, 9]
      //   console.log('fillinf')
      for (let i = 0; i < 81; i++) {
        row = Math.floor(i / 9)
        col = i % 9
        // console.log(row, col)
        if (this.completeGrid[row][col] === 0) {
          numList = this.shuffleList(numList)
          for (const i in numList) {
            const num = numList[i]
            // console.log(num)
            // check num not already in row
            if (!this.completeGrid[row].includes(num)) {
              //   console.log('ha')
              // check num not in column
              if (!this.getGridColumn(this.completeGrid, col).includes(num)) {
                // check num not in current sub grid

                if (
                  !this.subGridValues(this.completeGrid, row, col).includes(num)
                ) {
                  console.log(row, col, num)
                  this.completeGrid[row][col] = num
                  if (this.checkGrid(this.completeGrid)) {
                    return true
                  } else if (this.fillGrid()) {
                    return true
                  }
                }
              }
            }
          }

          break
        }
      }
      this.completeGrid[row][col] = 0
    },
    solve(grid) {
      let row, col

      for (let i = 0; i < 81; i++) {
        row = Math.floor(i / 9)
        col = i % 9
        if (grid[row][col] === 0) {
          for (let num = 1; num < 10; num++) {
            if (!grid[row].includes(num)) {
              if (!this.getGridColumn(grid, col).includes(num)) {
                if (!this.subGridValues(grid, row, col).includes(num)) {
                  grid[row][col] = num
                  if (this.checkGrid(grid)) {
                    this.solutions++
                    break
                  } else if (this.solve(grid)) {
                    return true
                  }
                }
              }
            }
          }
          break
        }
      }
      grid[row][col] = 0
    },
    // removes cells from display grid making sure it is still solvable
    removeCells(attempts) {
      // start removing numbers one by one
      // higher attempts will end up with removing more number
      do {
        // select a random non empty cell
        let row = this.rng(0, 8)
        let col = this.rng(0, 8)
        while (this.displayGrid[row][col] === 0) {
          row = this.rng(0, 8)
          col = this.rng(0, 8)
        }

        // backup cell value in case it needs to be put back
        const backup = this.displayGrid[row][col]
        this.displayGrid[row][col] = 0

        // count number of solution grid has using backtracking in solve function
        const gridCopy = this.displayGrid.slice()
        this.solutions = 0
        this.solve(gridCopy)

        // if the solution is not one put the value back in grid
        if (this.solutions !== 1) {
          this.displayGrid[row][col] = backup
          attempts--
        }
      } while (attempts > 0)
    },
    getGridColumn(grid, col) {
      // returns all values in a specific column
      return grid.map((row) => row[col])
    },
    subGridValues(grid, row, col) {
      // find subgrid and return all values within that subgrid
      for (const sub in this.subGrids) {
        const subGrid = this.subGrids[sub]
        for (let i = 0; i < subGrid.length; i++) {
          const [gridRow, gridCol] = subGrid[i]
          if (gridRow === row && gridCol === col) {
            return subGrid.map((pos) => grid[pos[0]][pos[1]])
          }
        }
      }
      return []
    },
    shuffleList(a) {
      for (let i = a.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1))
        ;[a[i], a[j]] = [a[j], a[i]]
      }
      return a
    },
    rng(min, max) {
      return Math.floor(Math.random() * (max - min + 1)) + min
    },
  },
}
</script>

<style lang="scss" scoped></style>
