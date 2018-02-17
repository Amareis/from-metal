<template>
  <div v-if="!compactView" class="number">
    <one-digit class="size" :value="sizeDigit"></one-digit>
    <one-digit v-for="(n, i) in valueDigits" :value="n" :key="i"></one-digit>
  </div>
  <span v-else class="compact">{{asText}}</span>
</template>

<script>
module.exports = {
	props: {
		digits: {
			type: Array,
			required: true,
      validator: arr => arr.every(d => d.constructor === Number && d >= 0 && d <= 9)
		},
		compactView: {
			type: Boolean,
			default: true,
		},
	},
	data: () => ({
	}),
  computed: {
    sizeDigit() {
    	return this.digits[0]
    },
    valueDigits() {
    	return this.digits.slice(1)
    },
    asText() {
    	return this.valueDigits.join('')
    },
  }
}
</script>

<style>
.number {
	display: inline-flex;
}
.size {
	background-color: gray;
}
.compact {
  font-family: monospace;
  font-size: 20px;
  border: 1px solid gray;
}
</style>