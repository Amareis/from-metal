<template>
    <div contenteditable="true" @input="update($event.target.value)"></div>
</template>

<script>
module.exports = {
  name: "span-editable",
  props: ['value'],
  mounted() {
    this.$el.innerText = this.value;
  },
  methods:{
    update(value){
      let formattedValue = value
        .trim()
        .slice(
          0,
          value.indexOf('.') === -1
            ? value.length
            : value.indexOf('.') + 3
        )
      // Если значение не нормализовано — нормализуем вручную
      if (formattedValue !== value) {
        this.$refs.div.value = formattedValue
      }
      // Порождаем событие с обновлённым значением поля ввода
      this.$emit('input', Number(formattedValue))
    }
  }
}
</script>

<style scoped>

</style>