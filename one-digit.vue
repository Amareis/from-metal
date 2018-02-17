<template>
	<span class="digit">{{value}}</span>
</template>

<script>
module.exports = {
	props: {
		value: {
			type: Number,
			required: true,
      validator: value => value >= 0 && value < 10
		}
	},
	data: () => ({
	}),
}
</script>

<style scoped>
.digit {
  font-family: monospace;
  font-size: 20px;
  width: 25px;
  text-align: center;
}
</style>