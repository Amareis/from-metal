<template>
  <span class="number">{{value}}</span>
</template>

<script>
module.exports = {
  props: {
    value: {
      type: Number,
      required: true,
    },
  },
}
</script>

<style scoped>
.number {
  font-family: monospace;
  font-size: 20px;
  text-align: center;
  padding: 0 5px;

}
</style>