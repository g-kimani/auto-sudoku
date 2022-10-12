<template>
  <div :key="key" class="game-grid">
    <v-row
      v-for="(row, x) in displayGrid"
      :key="x + row.length"
      no-gutters
      class="d-flex fill-height"
    >
      <v-col v-for="(cell, y) in row" :key="cell.id">
        <!-- {{ cell }} -->
        <slot name="cell" :value="cell.value" :x="x" :y="y"></slot>
      </v-col>
    </v-row>
  </div>
</template>

<script>
export default {
  props: {
    grid: { type: Array, required: true },
  },
  data() {
    return {
      key: 0,
    }
  },
  computed: {
    displayGrid() {
      let id = 0
      const cells = []
      for (let row = 0; row < 9; row++) {
        const store = []
        for (let col = 0; col < 9; col++) {
          console.log(id, this.grid[row][col])
          id++
          store.push({ id, value: this.grid[row][col] })
        }
        cells.push(store)
      }
      return cells
    },
  },
  methods: {
    updateScreen() {
      this.key++
    },
  },
}
</script>

<style lang="scss" scoped>
.game-grid {
  border: 1px solid black;
  border-radius: 6px;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 400px;
  position: relative;
  min-width: 480px;
}
// sub dividing the grid
.game-grid .row:nth-child(3n) {
  border-bottom: 3px solid black;
}

.game-grid .row .col:nth-child(3n) {
  border-right: 3px solid black;
}
</style>
