<template>
  <div>
    <grid-display ref="grid" :key="gridKey" :grid="activeGrid" @keypress="log">
      <template #cell="{ value, x, y }">
        <grid-cell
          :value="value"
          :shaded="shadedCell(x, y)"
          @click="selectCell(x, y)"
        />
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
      solvedGrid: new Array(9).fill(0).map(() => new Array(9).fill(0)),
      playerGrid: new Array(9).fill(0).map(() => new Array(9).fill(0)),
      // 3 * 3 grid for each cell
      notesGrid: new Array(27).fill(0).map(() => new Array(27).fill(0)),
      displayGrid: [],
      subGrids: {},
      showWarnings: false,
      solutions: 0,
      selectedCell: null,
      shadedCells: [],
      emptyCells: 0,
      gridKey: 0,
    }
  },
  computed: {
    activeGrid() {
      if (this.displayGrid.length === 0) return []
      let x = -1
      return this.displayGrid.map((row) => {
        x++
        let y = -1
        return row.map((cell) => {
          y++
          if (cell === 0) {
            return this.playerGrid[x][y]
          } else {
            return cell
          }
        })
      })
    },
  },
  watch: {
    selectedCell(value) {
      console.log(value)
    },
  },
  created() {
    this.divideSubGrids()
    this.fillGrid()
    this.displayGrid = this.cloneObject(this.solvedGrid)
    this.removeCells(1)
  },
  mounted() {
    window.addEventListener('keydown', (e) => {
      // registers only key downs for 1 - 9 on either numberpad or keyboard
      // eslint-disable-next-line eqeqeq
      if ([...Array(9).keys()].findIndex((k) => k + 1 == e.key) !== -1) {
        if (this.selectedCell) {
          console.log('adding number', e.key)
          this.addNumber(e.key)
        }
      }
      if (e.key === 'Backspace' || e.key === 'Delete') {
        this.eraseCell()
      }
    })
    this.emptyCells = this.displayGrid.reduce((acc, row) => {
      return acc + row.filter((r) => r === 0).length
    }, 0)
  },
  methods: {
    log(x, y, value) {
      console.log(x, y, value)
    },
    checkWin() {
      return this.solvedGrid.every((row, x) =>
        row.every((cell, y) => {
          console.log(x, y, cell)
          return (
            this.displayGrid[x][y] === cell || this.playerGrid[x][y] === cell
          )
        })
      )
    },
    selectCell(x, y) {
      this.selectedCell = { x, y }
      this.shadeCells()
    },
    shadeCells() {
      this.shadedCells = []
      if (!this.selectedCell) return
      const x = this.selectedCell.x
      const y = this.selectedCell.y
      this.shadedCells.push(this.selectedCell)
      for (let offY = 0; offY < this.solvedGrid.length; offY++) {
        this.shadedCells.push([x, offY])
      }
      this.getGridColumn(this.solvedGrid, y).forEach((pos) =>
        this.shadedCells.push(pos)
      )
      this.getSubGrid(x, y).forEach((pos) => {
        this.shadedCells.push(pos)
      })

      let cellValue = this.displayGrid[x][y]
      if (cellValue === 0) {
        cellValue = this.playerGrid[x][y]
      }

      for (let row = 0; row < this.solvedGrid.length; row++) {
        for (let col = 0; col < this.solvedGrid[row].length; col++) {
          if (row !== x && col !== y) {
            if (
              (this.displayGrid[row][col] === cellValue ||
                this.playerGrid[row][col] === cellValue) &&
              cellValue !== 0
            ) {
              this.shadedCells.push([row, col])
            }
          }
        }
      }
    },
    shadedCell(x, y) {
      return (
        this.shadedCells.findIndex((pos) => pos[0] === x && pos[1] === y) !== -1
      )
    },
    addNumber(value) {
      const row = this.selectedCell.x
      const col = this.selectedCell.y
      console.log(row, col, value)
      if (this.displayGrid[row][col] === 0) {
        console.log(value)
        this.$set(this.playerGrid[row], col, parseInt(value))
        console.log(this.playerGrid[row][col])
        if (this.solvedGrid[row][col] === value) {
          this.emptyCells--
        }
        if (this.checkWin()) {
          console.log('won: ', this.checkWin())
        }
      }
    },
    eraseCell() {
      const row = this.selectedCell.x
      const col = this.selectedCell.y
      this.$set(this.playerGrid[row], col, 0)
    },
    divideSubGrids() {
      // divides 9*9 grid into 3*3 sub grids and stores their positions

      let row = 1 // start at 1 to be in middle of sub grid
      let col = 1

      let subGrid = 1
      for (let i = 0; i < 3; i++) {
        for (let j = 0; j < 3; j++) {
          // initialise array to store positions
          this.subGrids[subGrid] = []
          // loop over all adjacent cells starting a center of sub grid and store
          for (let x = -1; x < 2; x++) {
            for (let y = -1; y < 2; y++) {
              this.subGrids[subGrid].push([row + x, col + y])
            }
          }
          subGrid++ // move to next sub grid
          col += 3 // move to middle of next sub grid in the row
        }
        row += 3 // move to middle of next sub grid in column, starting a new row
        col = 1 // reset col to middle of first subgrid in new row
      }
    },
    gridFilled(grid) {
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
      for (let i = 0; i < 81; i++) {
        row = Math.floor(i / 9)
        col = i % 9
        if (this.solvedGrid[row][col] === 0) {
          numList = this.shuffleList(numList)
          for (const i in numList) {
            const num = numList[i]
            // check num not already in row
            if (!this.solvedGrid[row].includes(num)) {
              // check num not in column
              if (
                !this.getValues(
                  this.solvedGrid,
                  this.getGridColumn(this.solvedGrid, col)
                ).includes(num)
              ) {
                // check num not in current sub grid
                if (
                  !this.getValues(
                    this.solvedGrid,
                    this.getSubGrid(row, col)
                  ).includes(num)
                ) {
                  this.$set(this.solvedGrid[row], col, num)
                  if (this.gridFilled(this.solvedGrid)) {
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
      this.$set(this.solvedGrid[row], col, 0)
    },
    solve(grid) {
      let row, col

      for (let i = 0; i < 81; i++) {
        row = Math.floor(i / 9)
        col = i % 9
        if (grid[row][col] === 0) {
          for (let num = 1; num < 10; num++) {
            if (!grid[row].includes(num)) {
              if (
                !this.getValues(grid, this.getGridColumn(grid, col)).includes(
                  num
                )
              ) {
                if (
                  !this.getValues(grid, this.getSubGrid(row, col)).includes(num)
                ) {
                  grid[row][col] = num
                  if (this.gridFilled(grid)) {
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
        this.$set(this.displayGrid[row], col, 0)

        // count number of solution grid has using backtracking in solve function
        const gridCopy = this.cloneObject(this.displayGrid)
        this.solutions = 0
        this.solve(gridCopy)

        // if the solution is not one put the value back in grid
        if (this.solutions !== 1) {
          this.$set(this.displayGrid[row], col, backup)
          attempts--
        }
      } while (attempts > 0)
    },
    getGridColumn(grid, col) {
      // returns all positions in a specific column
      return grid.map((row) => [grid.indexOf(row), col])
    },
    getSubGrid(row, col) {
      for (const sub in this.subGrids) {
        const subGrid = this.subGrids[sub]
        for (const pos in subGrid) {
          const [gridRow, gridCol] = subGrid[pos]
          if (gridRow === row && gridCol === col) {
            return subGrid
          }
        }
      }
    },
    getValues(grid, positions) {
      return positions.map((pos) => grid[pos[0]][pos[1]])
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
    cloneObject(obj) {
      return JSON.parse(JSON.stringify(obj))
    },
  },
}
</script>

<style lang="scss" scoped></style>
