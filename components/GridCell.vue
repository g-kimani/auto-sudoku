<template>
  <v-card :class="classList" tile @click="$emit('click')">
    {{ value === 0 ? '' : value }}
  </v-card>
</template>

<script>
export default {
  props: {
    value: { type: Number, default: 0 },
    shaded: { type: Boolean, default: false },
  },
  computed: {
    classList() {
      const classes = ['fill-height', 'grid-cell']
      if (this.shaded) {
        classes.push('shaded')
      }
      return classes
    },
  },
}
</script>

<style lang="scss" scoped>
.shaded {
  background-color: gray !important;
}
.grid-cell {
  display: flex;
  justify-content: center;
  align-items: center;
  min-width: 20px;
  min-height: 20px;
  width: 100%;
  height: 100%;
  border: 1px solid rgb(0, 0, 0);
  font-weight: 700;
  background-color: whitesmoke;
  color: black;
}
</style>
