<template>
  <div>
    <textarea v-model="rawMemory" placeholder="New memory"></textarea>
    <button @click="newMemory">Set memory</button> <br>

    <input id="divOps" type="checkbox" v-model="divideOperations" />
    <label for="divOps">Divide operations</label>
    <br><br>

    <div class="ops">
      <template v-for="(numbers, opi) in ops">
        <one-number
          v-for="(number, ni) in numbers"
          v-model="numbers[ni]"
          :key="opi + ' ' + ni"
          :class="{active: isActive(opi, ni)}"
        ></one-number>
        <div v-if="divideOperations" class="divider"></div>
      </template>
    </div>
    <br>
    <button @click="step">One step</button>
    <br><br>
    <br><br>
    <button @click="newMemory">Set rules</button> <br> <br>
    <textarea v-model="rawRules" placeholder="Rules"></textarea>
    <div v-for="rule of rules" class="rule">{{rule}}</div>
  </div>
</template>

<script>
const rawRules = `
0   nop
1   IP = $1
10  if (D == 0) IP = $1
2   *1 = *1 + 1
20  *1 = *1 + $2
21  *1 = *1 + *2
`

const proc = {
  opBad() {
    throw new Error('Bad opcode!')
  },

  op0() {},

  op1(arg) {
    this.nextOpAddr = arg
  },
  op10(arg) {
    if (this.diff === 0)
      this.nextOpAddr = arg
  },
  op100(arg) {
    if (this.diff !== 0)
      this.nextOpAddr = arg
  },
  op11(arg) {
    if (this.diff < 0)
      this.nextOpAddr = arg
  },
  op12(arg) {
    if (this.diff > 0)
      this.nextOpAddr = arg
  },
  op13(arg) {
    if (this.diff <= 0)
      this.nextOpAddr = arg
  },
  op14(arg) {
    if (this.diff >= 0)
      this.nextOpAddr = arg
  },
  op15(arg) {
    if (this.diff % 2 === 0)
      this.nextOpAddr = arg
  },
  op16(arg) {
    if (this.diff % 2 !== 0)
      this.nextOpAddr = arg
  },

  op2(arg) {
    this.set(arg, this.get(arg) + 1)
  },
  op20(arg1, arg2) {
    this.set(arg1, this.get(arg1) + arg2)
  },
  op21(arg1, arg2) {
    this.set(arg1, this.get(arg1) + this.get(arg2))
  },

  op3(arg) {
    this.set(arg, this.get(arg) - 1)
  },
  op30(arg1, arg2) {
    this.set(arg1, this.get(arg1) - arg2)
  },
  op31(arg1, arg2) {
    this.set(arg1, this.get(arg1) - this.get(arg2))
  },

  op4(arg1, arg2) {
    this.set(arg1, this.get(arg2))
  },

  op40(arg1, arg2) {
    this.set(arg1, this.get(arg1) * arg2)
  },
  op41(arg1, arg2) {
    this.set(arg1, this.get(arg1) * this.get(arg2))
  },

  op50(arg1, arg2) {
    this.set(arg1, Math.floor(this.get(arg1) / arg2))
  },
  op51(arg1, arg2) {
    this.set(arg1, Math.floor(this.get(arg1) / this.get(arg2)))
  },

  op60(arg1, arg2) {
    this.set(arg1, this.get(arg1) % arg2)
  },
  op61(arg1, arg2) {
    this.set(arg1, this.get(arg1) % this.get(arg2))
  },

  op7(arg1) {
    this.diff = this.get(arg1)
  },
  op70(arg1, arg2) {
    this.diff = this.get(arg1) - arg2
  },
  op71(arg1, arg2) {
    this.diff = this.get(arg1) - this.get(arg2)
  },

  op8(arg) {
    alert(this.get(arg))
  },
  op9(arg) {
    let num
    do {
      num = parseInt(prompt())
    } while(isNaN(num))
    this.set(arg, num)
  }
}

const initMemory = [2,6, 2,6, 1,6, 0, 2,0, 1,0]

module.exports = {
  data() {
    return {
      nextOpAddr: 0,
      diff: 0,
      memory: initMemory,
      divideOperations: true,
      rawMemory: initMemory.join(' '),
      rawRules,
      rules: {},
    }
  },

  mounted() {
    this.newRules()
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

    newMemory() {
      this.memory = this.rawMemory.trim().split(/\s+/).map(Number)
      this.nextOpAddr = 0
    },

    newRules() {
      this.rawRules = this.rawRules.trim().replace(/\n+/, '\n')
      for (let rawRule of this.rawRules.split(/\n/)) {
        let [opcode, ...tokens] = rawRule.split(/\s+/)
        tokens = tokens.join(' ')
        this.rules[opcode] = tokens
          .replace('IP', 'this.nextOpAddr')
          .replace('D', 'this.diff')
          .replace(/(.*)=(.*)/, 'this.set($1, $2)')
          .replace(/\*(\d+)/g, 'this.get(arg$1)')
          .replace(/\$(\d+)/g, 'arg$1')
      }
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
  .rule {
    margin-top: 10px;
  }
</style>