<template>
  <div id="app">
    <div class="start-btn" @click="startGame">开始</div>
    <div class="total">总分：{{total}}</div>
    <game ref="game" @changeTotal="changeTotal"></game>
  </div>
</template>

<script>
import game from './views/game'

export default {
  name: 'App',
  components: {
    game
  },
  data() {
      return {
        total: 0,
      }
  },
  methods: {
    startGame() {
      this.$refs.game.start()
    },
    changeTotal (data) {
      this.total = data.reduce((pre, cur, index, arr) => {
        if (typeof pre == 'object') {
          return pre.value+ cur.value
        } else {
          return pre+ cur.value
        }
        
      })
    },
  },
}
</script>

<style scoped>
#app {overflow: hidden;}
.start-btn {
    margin: 0 auto;
    width: 100px;
    text-align: center;
    height: 25px;
    background: #4275e6;
    line-height: 25px;
    border-radius: 5px;
    color: #fff;
    margin-top: 25px;
}
.total {
  text-align: center;
  margin-top: 20px;
}
</style>