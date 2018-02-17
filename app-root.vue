<template>
  <div>
    <button @click="compactView = !compactView">Switch view</button>
    <br>
    <button @click="step">One step</button>
    <br><br>
    <one-number v-for="(digs, i) in digits" :key="i" :digits="digs" :compact-view="compactView"></one-number>
    <div v-if="!compactView" class="pointer" :style="{left: 25*this.nextOpAddr+'px'}">
  </div>
</template>

<script>
module.exports = {
  data() {
    return {
      compactView: false,
      nextOpAddr: 0,
      memory: [1,2, 2,1,5, 1,3, 2,1,5, 1,1, 1,0, 1,3],
    }
  },

  computed: {
    digits() {
      const res = []
      for (let i = 0; i < this.memory.length;) {
        const curDigits = this.digitsAtAddr(i)
        res.push(curDigits)
        i += curDigits[0] + 1
      }
      return res
    },
  },

  methods: {
    digitsAtAddr(addr) {
      const len = this.memory[addr]
      return this.memory.slice(addr, addr + len + 1)
    },
    numberAtAddr(addr) {
      return parseInt(this.digitsAtAddr(addr).slice(1).join(''))
    },
    nNumbersFromAddr(n, addr) {
      const res = []
      let len = 0
      while (n) {
        res.push(this.numberAtAddr(addr + len))
        len += this.memory[addr] + 1
        n--
      }
      return [res, len]
    },
    step() {
      const currOpAddr = this.nextOpAddr
      const op = this['op' + this.numberAtAddr(currOpAddr)]
      const argsStart = currOpAddr + this.memory[currOpAddr] + 1
      const [args, argsLen] = this.nNumbersFromAddr(op.length, argsStart)
      this.nextOpAddr = argsStart + argsLen
      op(...args)
    },

    op1(arg) {
      this.nextOpAddr = arg
    },
    op2(arg) {
      Vue.set(this.memory, arg, this.memory[arg] + 1)
    },
    op3(arg) {
      Vue.set(this.memory, arg, this.memory[arg] - 1)
    },
  },
}
</script>

<style scoped>
.pointer {
  position: relative;
  width: 25px;
  height: 25px;
  top: -24px;
  background-color: red;
  opacity: 0.5;
}
</style> 