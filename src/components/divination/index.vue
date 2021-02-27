<template>
  <div class="divination">
    <div class="d-header">
      <div class="left">
        <span class="btn"></span>
      </div>
      <div class="center">
        <span class="logo"></span>
      </div>
      <div class="right">
        <span class="btn bl"></span>
        <span class="btn br"></span>
      </div>
    </div>
    <div class="d-log">
      <div class="log-label">
        开牌记录
      </div>
      <div class="log-list">
        <span class="item" v-for="(item, index) of logListData"  :key="index">
          <img :src="getItemIconUrl(item.type)" alt="">
        </span>
        <span class="item">
          <img src="./assets/arrow.png" style="height: 12px;width: auto;vertical-align: middle;margin-top: -2px;" alt="">
        </span>
      </div>
    </div>
    <div class="d-title">
      <span class="title-box">
        {{title}}
      </span>
    </div>
    <div class="d-content">
      <div class="box" :style="{left: moveCard ? '5%' : 0 }">
        <div class="item" @click="betting(index)" :class="{animation: opening, opened: opened }" :style="{top: isNeedMove(item) ? itemHeight + 'px' : 0, left: isNeedMove(item) ? '-12%' : 0 }" :ref="index" v-for="(item, index) of iconData" :key="index">
          <div class="num" :class="{show: !opening && coinData['x'+item.multiple]}">共{{coinData['x'+item.multiple]}}</div>
          <div class="zm" :class="{opening: opening}">
            <img src="./assets/bg-card_fa5cd69e.png" style="width: 100%" alt="">
            <img :src="item.icon" class="icon" :class="{tk: item.multiple === 60}" alt="">
            <div class="text">{{item.multiple}}倍</div>
            <div class="label">{{item.label}}</div>
          </div>
          <div class="bm" :class="{opening: opening}">
            <img src="./assets/card-back_937dd978.png" style="width: 93%" alt="">
          </div>
        </div>
        <span v-if="!opened" class="cursor-icon" ref="cursor-icon"></span>
      </div>
      <div v-if="opened && result" class="result">
        <div class="zm">
          <img src="./assets/bg-card_fa5cd69e.png" style="width: 100%" alt="">
          <img :src="iconData[result].icon" class="icon" :class="{tk: iconData[result].multiple === 60}" alt="">
          <div class="text">{{iconData[result].multiple}}倍</div>
          <div class="label">{{iconData[result].label}}</div>
        </div>
        <div class="bm">
          <img src="./assets/card-back_937dd978.png" style="width: 93%" alt="">
        </div>
      </div>
    </div>
    <div class="d-bottom">
      <div class="coins">
        <span class="coin" :class="{settinging: item === bettingNum}" v-for="item of addCoinData" :key="item" @click="setBettingNum(item)">{{item}}</span>
      </div>
      <div class="mycoin">
        {{myCoinNum}}
      </div>
    </div>
  </div>
</template>

<script>
  // 不同牌对应的icon和文字关系
  const iconData = {
    x2: {
      icon: require('./assets/x2.png'),
      multiple: 2,
      label: '幸运星'
    },
    x3: {
      icon: require('./assets/x3.png'),
      multiple: 3,
      label: '创可贴'
    },
    x5: {
      icon: require('./assets/x5.png'),
      multiple: 5,
      label: '脸谱'
    },
    x10: {
      icon: require('./assets/x10.png'),
      multiple: 10,
      label: '超级跑车'
    },
    x20: {
      icon: require('./assets/x20.png'),
      multiple: 20,
      label: '马戏团'
    },
    x40: {
      icon: require('./assets/x40.png'),
      multiple: 40,
      label: '摩天轮'
    },
    x60: {
      icon: require('./assets/x60.png'),
      multiple: 60,
      label: '私奔到太空'
    },
    x80: {
      icon: require('./assets/x80.png'),
      multiple: 80,
      label: '梦幻城堡'
    },
  }
  const addCoinData = [10, 50, 100, 500]
  export default {
    name: "divination",
    props: {
      title: String, // 标题
      coinData: {
        type: Object,
        default: () => {
          return {}
        }
      }, // 下注金额数据
      logListData: Array, // 开奖记录列表
      myCoinNum: {
        default: 0
      }, // 我的金币余额
    },
    data() {
      return {
        iconData,
        time: 0,
        status: 0,
        beforeOpen: false,
        opening: false,
        opened: false,
        itemHeight: 123,
        cursor: '',
        moveCard: false,
        moveCarded: false,
        result: '', // 开奖结果，枚举值 x2，x5....
        // eslint-disable-next-line vue/no-dupe-keys
        addCoinData,
        bettingNum: addCoinData[0]
      }
    },
    computed: {
    },
    watch: {
      moveCarded(val) {
        if(!val) {
          this.cursor = ''
        } else {
          this.randomCursor()
        }
      }
    },
    created() {
    },
    mounted() {
    },
    beforeDestroy() {
      clearInterval(this.interval)
    },
    methods: {
      start() {
        this.getItemHeight()
        this.beforeOpen = true
      },
      wait() {
        this.beforeOpen = false
        this.opening = true
        this.getItemHeight()
        setTimeout(() => {
          this.moveCard = true
        }, 1000)
        setTimeout(() => {
          this.moveCarded = true
        }, 2000)
      },
      end(result) {
        this.result = result
        clearInterval(this.interval)
        setTimeout(() => {
          this.setOpenedCard()
        }, 800)
      },
      betting(type) {
        if (!this.beforeOpen) {
          return
        }
        this.$emit('betting', {
          type,
          num: this.bettingNum
        })
      },
      getItemIconUrl(type) {
        return iconData[`x${type}`].icon
      },
      isNeedMove(item) {
        return item.multiple <= 10 && this.moveCard // 上面四张卡牌需要移动位置
      },
      getItemHeight() {
        // console.log(this.$refs[key]);
        if (this.$refs['x2']) {
          this.itemHeight = this.$refs['x2'][0].offsetHeight
        }
      },
      randomCursor() {
        let index = -1
        this.interval = setInterval(() => {
          index++
          if(index >= 8) {
            clearInterval(this.interval)
            this.randomCursor2()
          } else {
            this.moveCardTop(index, 200, 30)
          }
        }, 200)
      },
      randomCursor2() {
        let index = 0
        this.interval = setInterval(() => {
          index = Math.floor(Math.random() * 8)
          this.moveCardTop(index, 800, 40)
        }, 800)
      },
      moveCardTop(index, time, top) {
        if (this.opened) {
          return
        }
        const data = ['x2', 'x20', 'x3', 'x40', 'x5', 'x60', 'x10', 'x80']
        this.setCursorIconPosition(index)
        const i = data[index]
        this.cursor = i
        const topnow = this.isNeedMove(this.iconData[i]) ? this.itemHeight : '0'
        const left = this.isNeedMove(this.iconData[i]) ? '-12' : '0'
        const position = this.isNeedMove(this.iconData[i]) ? 'position: relative;' : ''
        this.$refs[i][0].style = `position: relative;top:${topnow - top}px;left:${left}%;`
        setTimeout(() => {
          this.$refs[i][0].style = `${position}top:${topnow}px;left:${left}%;`
        }, time)
      },
      setCursorIconPosition(index) {
        this.$refs['cursor-icon'].style = `left:${index * 12}%`
      },
      setOpenedCard() {
        const needMove = this.isNeedMove(this.iconData[this.cursor])
        const top = needMove ? 0 : -1 * this.itemHeight
        const left = needMove ? '-12' : '0'
        this.$refs[this.cursor][0].style = `top: ${top}px;left:${left}%;`
        setTimeout(() => {
          this.$refs[this.cursor][0].style = `top: ${top}px;left:${left}%;opacity:0;`
          this.opened = true
        }, 200)
      },
      setBettingNum(num) {
        this.bettingNum = num
      }
    },
  }
</script>

<style scoped>
  .divination * {
    background-size: contain;
    background-repeat: no-repeat;
  }
  .divination {
    position: fixed;
    bottom: 0;
    left: 0;
    background-image: url("./assets/bg-draw_4929485d.png");
    background-size: cover;
    width: 100%;
  }
  .d-header {
    display: flex;
    padding: 0 9px;
    line-height: 0px;
  }
  .d-header>div {
    flex: 1;
    padding: 9px 0;
  }
  .d-header .left .btn {
    display: inline-block;
    width: 60px;
    height: 24px;
    background-image: url("./assets/btn-rank_43a21544.png");
  }
  .d-header .center {
    padding: 0;
  }
  .d-header .center .logo {
    display: inline-block;
    width: 145px;
    height: 39px;
    background-image: url("./assets/title_f0c13f04.png");
  }
  .d-header .right {
    text-align: right;
  }
  .d-header .right .btn {
    display: inline-block;
    width: 44px;
    height: 24px;
  }
  .d-header .right .btn.bl {
    background-image: url("./assets/btn-rule_e6af3dd0.png");
  }
  .d-header .right .btn.br {
    background-image: url("./assets/btn-record_a8d15fe4.png");
  }
  .d-log{
    margin: 0 9px;
    height: 24px;
    border-radius: 12px;
    background-color: #4901aa;
    font-size: 12px;
    line-height: 24px;
    padding: 0 10px;
    color: #fff3fd;
    display: flex;
  }
  .log-label {
    flex: 1;
    text-align: center;
  }
  .log-list {
    flex: 4;
    display: flex;
    width: calc(100% - 100px);
    line-height: 24px;
    height: 24px;
  }
  .log-list .item {
    flex: 1;
  }
  .log-list img{
    height: 18px;
    width: 18px;
    vertical-align: text-bottom;
  }
  .d-title {
    margin-top: 9px;
    text-align: center;
  }
  .title-box {
    display: inline-block;
    width: 167px;
    height: 42px;
    line-height: 42px;
    font-size: 16px;
    color: #fff;
    background-image: url("./assets/plate-title_dd88a792.png");
  }
  .d-content {
    position: relative;
    padding: 10px 19px;
  }
  .d-content .box {
    display: flex;
    align-content: flex-start;
    position: relative;
    flex-flow: row wrap;
    justify-content:space-evenly;
    transition: left 0.4s;
  }
  .d-content .box .item {
    flex: 0 0 19vw;
    transform-style: preserve-3d;
  }
  .d-content .box .item .zm {
    position: relative;
  }
  .d-content .box .item .bm {
    opacity: 0;
    transform: translate3d(0, 0, -1px);
    position: absolute;
    top: 20px;
    left: 0;
  }
  .d-content .box .item .zm.opening {
    opacity: 0;
    transition: all 1s;
  }
  .d-content .box .item .bm.opening {
    opacity: 1;
    transition: all 0.5s;
  }
  .d-content .box .item .num.show {
    opacity: 1;
  }
  .d-content .box .item .num {
    opacity: 0;
    max-width: calc(19vw - 22px);
    height: 16px;
    line-height: 16px;
    padding-left: 5px;
    padding-right: 10px;
    margin: 0 5px 3px;
    color: #ffffff;
    font-size: 12px;
    border-radius: 8px;
    background-color: #5e09cf;
    position: relative;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
  .d-content .box .item .num:after {
    content: '';
    width: 10px;
    height: 10px;
    display: inline-block;
    background-size: contain;
    background-image: url("./assets/icon-coin_83bd22c5.png");
    position: absolute;
    right: 3px;
    top: 3px;
  }
  .d-content .box .item .text {
    position: absolute;
    top: 2px;
    left: 0px;
    width: 19vw;
    text-align: center;
    color: #ffffff;
    font-size: 12px;
  }
  .d-content .box .item .label {
    position: absolute;
    bottom: 9px;
    left: 0px;
    width: 19vw;
    text-align: center;
    color: #ffffff;
    font-size: 12px;
  }
  .icon {
    width: 50%;
    position: absolute;
    left: 25%;
    top: 25%;
  }
  .icon.tk {
    width: 80%;
    left: 10%;
    top: 33%;
  }
  .animation {
    animation: cardback ease 0.7s 1 alternate;
    animation-fill-mode: forwards;
  }
  .item {
    transition: all 0.5s;
    position: relative;
  }
  .d-content .box .item:nth-child(1) {
    z-index: 10;
  }
  .d-content .box .item:nth-child(5) {
    z-index: 11;
  }
  .d-content .box .item:nth-child(2) {
    z-index: 12;
  }
  .d-content .box .item:nth-child(6) {
    z-index: 13;
  }
  .d-content .box .item:nth-child(3) {
    z-index: 14;
  }
  .d-content .box .item:nth-child(7) {
    z-index: 15;
  }
  .d-content .box .item:nth-child(4) {
    z-index: 16;
  }
  .d-content .box .item:nth-child(8) {
    z-index: 17;
  }
  .cursor.opened {
    top: 0!important;
  }
  .cursor {
    top: 80px!important;
  }
  .cursor.opened:after {
    opacity: 0;
  }
  .cursor:after {
    content: '';
    color: #fff3fd;
    width: 14px;
    height: 14px;
    display: inline-block;
    background-image: url("./assets/icon-cursor_f054453c.png");
    background-size: contain;
    background-repeat: no-repeat;
    position: absolute;
    left: calc(50% - 9px);
    bottom: -10px;
  }
  .cursor-icon {
    width: 14px;
    height: 14px;
    display: inline-block;
    background-image: url("./assets/icon-cursor_f054453c.png");
    position: absolute;
    left: -1000%;
    bottom: -10px;
    transition: all 0.2s;
  }
  .result {
    position: absolute;
    top: 9px;
    left: 50%;
    width: 100px;
    height: 133px;
    transform: translateX(-50%)  rotateY(180deg);
    display: inline-block;
    z-index: 999;
    transform-style: preserve-3d;
    animation: cardbackun ease 0.7s 1 alternate;
    animation-fill-mode: forwards;
  }
  .result .bm{
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    height: 100%;
    transform: translate3d(0,0,-1px);
    opacity: 0;
    transition: all 0.3s;
  }
  .result .text, .result .label {
    position: absolute;
    width: 100%;
    height: 23px;
    text-align: center;
    left: 0;
    color: #fff3fd;
  }
  .result .text {
    top: 2px;
  }
  .result .label {
    bottom: 0;
  }
  .d-bottom {
    margin-top: 10px;
    padding: 5px 0px;
    display: flex;
    background-color: #2d005d;
  }
  .d-bottom .coins {
    flex: 8;
    display: flex;
    justify-content:space-evenly;
  }
  .d-bottom .coins .coin {
    flex: 0 0 22%;
    text-align: left;
    text-indent: 26px;
    position: relative;
    height: 40px;
    line-height: 40px;
    color: #fff3fd;
    background-size: 100% 100%;
    background-image: url("./assets/btn-unsel_2daed365.png");
  }
  .coin:after, .mycoin:after {
    content: '';
    width: 16px;
    height: 16px;
    display: inline-block;
    background-size: contain;
    background-image: url("./assets/icon-coin_83bd22c5.png");
    position: absolute;
    left: 6px;
    top: 12px;
  }
  .settinging.coin {
    background-image: url("./assets/btn-sel_1e5fd82f.png")!important;
  }
  .mycoin {
    flex: 3;
    position: relative;
    margin: 0 9px 0 0;
    color: #fff3fd;
    background-color: #4c02ad;
    line-height: 40px;
    text-indent: 28px;
    border-radius: 20px;
  }
  .mycoin:before {
    content: '';
    width: 16px;
    height: 16px;
    display: inline-block;
    background-size: contain;
    background-image: url("./assets/icon-exchange_019fcb73.png");
    position: absolute;
    right: 6px;
    top: 12px;
  }
  /*.d-content .box .animation.item:nth-child(-n + 4){*/
  /*}*/
  /* Chrome, Safari, Opera */
  @-webkit-keyframes cardback {
    0% {
      transform: rotateY(0);
    }
    100% {
      transform: rotateY(180deg);
    }
  }

  /* Standard syntax */
  @keyframes cardback {
    0% {
      transform: rotateY(0);
    }
    100% {
      transform: rotateY(180deg);
    }
  }
  @keyframes cardbackun {
    0% {
      transform: translateX(-50%) rotateY(180deg);
    }
    100% {
      transform: translateX(-50%) rotateY(0);
    }
  }
  /* Standard syntax */
  @keyframes tiao {
    0% {
      transform: rotateY(0deg);
    }
    50% {
      top: 50px;
      transform: rotateY(0deg);
    }
    100% {
      transform: rotateY(0deg);
    }
  }
</style>
