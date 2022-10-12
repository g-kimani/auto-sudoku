<template>
  <v-card class="fill-height grid-cell" tile @click="$emit('click')">
    {{ value === 0 ? '' : value }}
  </v-card>
</template>

<script>
export default {
  props: {
    value: { type: Number, default: 0 },
  },
}
</script>

<style lang="scss" scoped>
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
