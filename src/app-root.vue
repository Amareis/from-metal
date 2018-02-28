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
    <!--button @click="newMemory">Set rules</button> <br> <br-->
    <textarea v-model="rawRules" placeholder="Rules"></textarea>
    <!--div v-for="(rule, opcode) of rules" class="rule">{{opcode}} {{rule}}</div-->
  </div>
</template>

<script>
const rawRules = `
0   nop

1   IP = $1
10  if (D == 0) IP = $1
100 if (D != 0) IP = $1
11  if (D < 0) IP = $1
12  if (D > 0) IP = $1
13  if (D <= 0) IP = $1
14  if (D >= 0) IP = $1
15  if (D % 2 == 0) IP = $1
16  if (D % 2 != 0) IP = $1

2   *1 = *1 + 1
20  *1 = *1 + $2
21  *1 = *1 + *2

3   *1 = *1 - 1
30  *1 = *1 - $2
31  *1 = *1 - *2

4   *1 = *2

40  *1 = *1 * $2
41  *1 = *1 * *2

50  *1 = Math.floor(*1 / $2)
51  *1 = Math.floor(*1 / *2)

60  *1 = *1 % $2
61  *1 = *1 % *2

7   D = *1
70  D = *1 - $2
71  D = *1 - *2

8   alert(*1)
9   *1 = parseInt(prompt()||0)
`

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
    opBad() {
      throw new Error('Bad opcode!')
    },
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
      return (this.rules[opCode] || this.opBad).bind(this)
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

    resetState() {
      this.nextOpAddr = 0
      this.diff = 0
    },

    newMemory() {
      this.resetState()
      this.memory = this.rawMemory.trim().split(/\s+/).map(Number)
    },

    newRules() {
      this.resetState()
      this.rawRules = this.rawRules.trim().replace(/\n+/, '\n')
      const rules = {}
      for (let rawRule of this.rawRules.split(/\n/)) {
        let [opcode, ...tokens] = rawRule.trim().split(/\s+/)
        tokens = tokens.join('')
        const argsNum = Math.max(...(tokens.match(/[$*]\d+/g) || ['00']).map(s => Number(s.slice(1))))
        tokens = tokens
          .replace(/nop/, '')
          .replace(/(.*)(IP|D)=(.*)/, '$1$2@EQ$3')
          .replace('IP', 'this.nextOpAddr')
          .replace('D', 'this.diff')
          .replace(/\*(\d+)=(.*)/, 'this.set($$$1, $2)')
          .replace(/@EQ/, '=')
          .replace(/\*(\d+)/g, 'this.get(arg$1)')
          .replace(/\$(\d+)/g, 'arg$1')
          .replace(/(.*)(if\([^)]*\))(.*)/g, '$2$1$3')
        rules[opcode] = Function(...[...Array(argsNum).keys()].map(n => 'arg' + (n+1)).concat([tokens]))
      }
      this.rules = rules
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