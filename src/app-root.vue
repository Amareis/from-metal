<template>
  <div>
    <input id="divOps" type="checkbox" v-model="divideOperations" />
    <label for="divOps">Divide operations</label>
    <br><br>
    <div class="ops">
      <template v-for="(numbers, opi) in ops">
        <one-number
          v-for="(number, ni) in numbers"
          :value="number"
          :key="opi + ' ' + ni"
          :class="{active: isActive(opi, ni)}"
        ></one-number>
        <div v-if="divideOperations" class="divider"></div>
      </template>
    </div>
    <br>
    <button @click="step">One step</button>
  </div>
</template>

<script>
const proc = {
  opBad() {
    throw new Error('Bad opcode!')
  },

  op0() {},

  op1(arg) {
    this.nextOpAddr = arg
  },
  op2(arg) {
    this.set(arg, this.get(arg) + 1)
  },
  op3(arg) {
    this.set(arg, this.get(arg) - 1)
  },
}

module.exports = {
  data() {
    return {
      nextOpAddr: 0,
      memory: [2,6, 2,6, 1,6, 0, 2,0, 1,0],
      divideOperations: true,
    }
  },

  computed: {
    ops() {
      const res = []
      for (let i = 0; i < this.memory.length;) {
        const opLen = this.opLenAt(i)
        res.push(this.getN(i, opLen))
        i += opLen
      }
      return res
    },
  },

  methods: {
    getN(addr, n) {
      return this.memory.slice(addr, addr + n)
    },
    get(addr) {
      return this.getN(addr, 1)[0]
    },
    set(addr, value) {
      Vue.set(this.memory, addr, value)
    },
    getOp(opCode){
      return (proc['op' + opCode] || proc.opBad).bind(this)
    },
    opLen(opCode){
      return this.getOp(opCode).length + 1
    },
    opLenAt(addr){
      return this.opLen(this.get(addr))
    },

    isActive(opI, nI) {
      let i = nI

      while (opI) {
        i += this.ops[opI - 1].length
        opI--
      }

      return this.nextOpAddr <= i && i < this.nextOpAddr + this.opLenAt(this.nextOpAddr)
    },

    step() {
      const opCode = this.get(this.nextOpAddr)
      const args = this.getN(this.nextOpAddr + 1, this.opLen(opCode) - 1)
      this.nextOpAddr += this.opLen(opCode)
      this.getOp(opCode)(...args)
    },
  },
}
</script>

<style scoped>
  .active {
    background-color: red;
  }
  .divider {
    width: 3px;
    background-color: black;
  }
  .ops {
    display: flex;
  }
</style>