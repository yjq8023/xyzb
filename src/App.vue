<template>
  <div id="app">
    <div style="text-align: center;padding-top: 20px;">
      <button @click="start" v-if="step === 0">开始</button>
      <button @click="wait" v-if="step === 1">下注结束</button>
      <button @click="end" v-if="step === 2">开奖</button>
      <button @click="init" v-if="step === 3">重置</button>
    </div>
    <divination
      :key="divinationKey"
      ref="divination"
      :title="title"
      :logListData="logListData"
      :coinData="coinData"
      :myCoinNum="998"
      @betting="betting"
    />
  </div>
</template>

<script>
import divination from './components/divination/index.vue'

export default {
  name: 'App',
  components: {
    divination
  },
  data() {
    return {
      logListData: [],
      title: '即将开始',
      coinData: {},
      reset: false,
      divinationKey: Date.now(),
      interval: null,
      step: 0
    }
  },
  mounted() {
    this.getLogListData()
  },
  methods: {
    // 初始化
    init() {
      this.step = 0
      this.divinationKey = Date.now()
      this.title = '即将开始'
      this.coinData = {}
    },
    // 开始下注
    start() {
      this.step = 1
      this.coinData = {}
      this.$refs.divination.start()
      let time = 0
      this.interval = setInterval(() => {
        time++
        this.title = `请许愿 ${40 - time}S`
        if (time >= 40) {
          time = 0
        }
      }, 1000)
    },
    // 结束下注，并播放开奖前的动画
    wait() {
      this.step = 2
      clearInterval(this.interval)
      this.title = '开奖中...'
      this.$refs.divination.wait()
    },
    // 开奖
    end() {
      this.step = 3
      this.title = '即将开始下一轮'
      this.$refs.divination.end('x2')
    },
    // 下注
    // data: {type: '下注类型（x倍）'， num: 下注金额}
    betting(data) {
      // 这里应该是实时获取所有人的下注金额，这里只是为了演示效果
      this.$set(this.coinData, data.type, this.coinData[data.type] ? this.coinData[data.type] + data.num : data.num)
    },
    // 获取开奖记录信息
    getLogListData() {
      // 模拟数据
      setTimeout(() => {
        // 以type为键，值是多少倍的意思
        this.logListData = [
          { type: 20 },
          { type: 2 },
          { type: 40 },
          { type: 3 },
          { type: 5 },
          { type: 10 },
          { type: 60 },
          { type: 80 }
        ]
      })
    }
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
}
#app {
  background-color: #e0e0e0;
  height: 100vh;
  width: 100vw;
}
</style>
