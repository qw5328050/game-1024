<template>
     <div id="game-background" ref="gameBg" @touchstart.stop.prevent="handleTouchStart" @touchend.stop.prevent="handleTouchEnd">
        <ul>
            <li class="game-item" v-for="(item, index) in bgList" :key="'bg'+index"></li>
        </ul>
        <template v-for="(item, index) in gameList">
            <template v-for="(item1, index1) in item">
                <div :class="`start-item start-item-${index + 1}_${index1 + 1} game-item_${item1}`" :key="'item' + index + index1" v-if="item1">{{item1}}</div>
            </template>
        </template>
    </div>
</template>

<script>
export default {
    name: "App",
    components: {},
    data() {
        return {
            bgList: [],
            gameList: [],
            gameStart: false,
            touchStart: {}
        }
    },

    // 监听属性 类似于data概念
    computed: {},
    // 监控data中的数据变化
    watch: {},
    // 生命周期 - 创建完成（可以访问当前this实例）
    created() {
        let that = this
        let label = 0
        for (let i = 1; i < 17; i++){
            if (i % 4 == 1) {
                label++
            }
            this.bgList.push({value: 0, pos: i, y: label, x: i % 4 == 0 ? 4 : i % 4})
        }
        // console.log(this.gameList)
        onkeydown = function (e) {
            var event = e || e.event;  // 标准化事件对象
            switch(event.keyCode){  // 获取当前按下键盘键的编码
                case 37 :  // 按下左箭头键，向左移动5个像素
                    that.move('left')
                    break;
                case 39 :  // 按下右箭头键，向右移动5个像素
                    that.move('right')
                    break;
                case 38 :  // 按下上箭头键，向上移动5个像素
                    that.move('up')
                    break;
                case 40 :  // 按下下箭头键，向下移动5个像素
                    that.move('down')
                    break;
            }
        }
    },
    // 生命周期 - 挂载完成（可以访问DOM元素）
    mounted() {},
    // 生命周期 - 创建之前
    beforeCreate() {},
    // 生命周期 - 挂载之前
    beforeMount() {},
    // 生命周期 - 更新之前
    beforeUpdate() {},
    // 生命周期 - 更新之后
    updated() {},
    // 生命周期 - 销毁之前
    beforeDestroy() {},
    // 生命周期 - 销毁完成
    destroyed() {},
    // 如果页面有keep-alive缓存功能，这个函数会触发
    activated() {},
    // 方法集合
    methods: {
        handleTouchmove (e) {
            console.log(e.touches[0])
        },
         handleTouchStart (e) {
             this.touchStart.screenX = e.changedTouches[0].screenX
             this.touchStart.screenY = e.changedTouches[0].screenY
        },
        handleTouchEnd (e) {
             let screenX = e.changedTouches[0].screenX
             let screenY = e.changedTouches[0].screenY
             let x = screenX - this.touchStart.screenX
             let y = screenY - this.touchStart.screenY
             let type = ''
             if (Math.abs(x) > Math.abs(y)) {
                 type = x > 0 ? "right" : 'left'
             } else {
                 type = y > 0 ? "down" : 'up'
             }
             this.move(type)
        },
        start () {
            this.gameStart = true
            this.initData()
            this.show()
        },
        initData () {
            for (let i = 0; i < 4; i++) {
                this.gameList[i] = []
                for (let j = 0; j < 4; j++) {
                    this.gameList[i][j] = 0
                }
            }
            this.bgList.map(v => {
                v.value = 0
            })
        },
        show () {
            let index = Math.ceil(Math.random() * 16) - 1
            let value = Math.ceil(Math.random() * 4)
            if (value == 3) {
                value = value - 2
            }
            if (!this.bgList[index].value) {
                this.addElement(value, index)
            } else {
                let data = this.bgList.filter((v, index) => {
                    return !v.value
                })
                if (data.length == 0) {
                    this.gameOver()
                    alert('game over')
                    return 
                }
                this.addElement(value, data[0].pos - 1)
            }
            this.$emit('changeTotal', this.bgList)
            console.log(JSON.stringify(this.gameList))
        },
        addElement (value, index) {
            let cur = this.bgList[index]
            let gameList = JSON.parse(JSON.stringify(this.gameList))
            cur.value = value
            gameList[cur.y - 1][cur.x - 1] = value
            this.gameList = gameList
        },
        move(type) {
            if (!this.gameStart) {
                return
            }
            let rule = {
                left: 'moveLeft',
                right: 'moveRight',
                up: 'moveUp',
                down: 'moveDown'
            }
            this[rule[type]]()
            this.handleBg()
            this.show()
        },
        handleBg () {
            this.bgList.map((v, i) => {
                let row = Math.floor(i/4);
                let col = i % 4;
                v.value = this.gameList[row][col]
                return v
            })
        },
        handleMove(type, gameList) {
            let rule = {
                down() {
                    for (let i = 0; i < 3; i++) {
                        gameList.forEach((data, y) => {
                            data.forEach((v, x) => {
                                if (y!=3 && gameList[y + 1][x] == 0&& gameList[y][x] > 0) {
                                    gameList[y + 1][x] = gameList[y][x]
                                    gameList[y][x] = 0
                                }
                            })
                        })
                    }
                },
                up() {
                    for (let i = 0; i < 3; i++) {
                        gameList.forEach((data, y) => {
                            data.forEach((v, x) => {
                                if (gameList[y][x] == 0 && y!=3 && gameList[y + 1][x] > 0) {
                                    gameList[y][x] = gameList[y + 1][x]
                                    gameList[y + 1][x] = 0
                                }
                            })
                        })
                    }
                },
                left() {
                    gameList.map(data => {
                        for (let idx = 0; idx < 3; idx++) {
                            data.forEach((v, i) => {
                                if (v == 0 && data[i+1]) {
                                    data[i] = data[i+1]
                                    data[i+1] = 0
                                }
                            })
                        }
                        return data
                    })
                },
                right() {
                    gameList.map(data => {
                        for (let idx = 0; idx < 3; idx++) {
                            data.forEach((v, i) => {
                                if (v == 0 && data[i-1]) {
                                    data[i] = data[i-1]
                                    data[i-1] = 0
                                }
                            })
                        }
                        return data
                    })
                }
            }
            rule[type]()
            return gameList
        },
        moveDown () {
            let gameList = JSON.parse(JSON.stringify(this.gameList))
            gameList = this.handleMove('down', gameList)
            for (let i = 3; i >= 0; i--) {
                let data = gameList[i]
                data.forEach((v, x) => {
                    if (i != 0 && gameList[i][x] == gameList[i - 1][x]) {
                        gameList[i][x] += gameList[i - 1][x]
                        gameList[i - 1][x] = 0
                    }
                })
            }
            gameList = this.handleMove('down', gameList)
            this.gameList = gameList
        },
        moveUp () {
            let gameList = JSON.parse(JSON.stringify(this.gameList))
            gameList = this.handleMove('up', gameList)
            gameList.forEach((data, i) => {
                data.forEach((v, x) => {
                    if (i != 3 && gameList[i][x] == gameList[i + 1][x]) {
                        let last = i + 2
                        gameList[i][x] += gameList[i + 1][x]
                        gameList[i + 1][x] = 0
                    }
                })
            })
            gameList = this.handleMove('up', gameList)
            this.gameList = gameList
        },
        moveLeft () {
            let gameList = JSON.parse(JSON.stringify(this.gameList))
            gameList = this.handleMove('left', gameList)
            gameList.map(data => {
                data.forEach((v,i) => {
                    if (data[i] == data[i + 1]) {
                        data[i] += data[i + 1]
                        data[i + 1] = 0
                    }
                })
                return data
            })
            gameList = this.handleMove('left', gameList)
            this.gameList = gameList
        },
        moveRight () {
            let gameList = JSON.parse(JSON.stringify(this.gameList))
            gameList = this.handleMove('right', gameList)
            gameList.map(data => {
                for(let i = 3; i >=0 ; i --) {
                    if (i != 0 && data[i] == data[i - 1]) {
                        data[i] += data[i - 1]
                        data[i - 1] = 0
                    }
                }
                return data
            })
            gameList = this.handleMove('right', gameList)
            this.gameList = gameList
        },
        gameOver () {
            this.gameStart = false
        }
    },
}
</script>
<style>
#game-background {
    background: #bdada0;
    width: 352px;
    height: 352px;
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    padding: 4px;
}
.game-item {
    width: 80px;
    height: 80px;
    margin: 4px;
    background: #ccbfb3;
    float: left;
}
.start-item {
    width: 80px;
    height: 80px;
    line-height: 80px;
    text-align: center;
    position: absolute;
    color: #fff;
}
.start-item.active {
    animation: move-up 1s;
}
@keyframes move-up {
	to { top: 0;}
}
.start-item-1_1 {
    top: 8px;
    left: 8px;
}
.start-item-1_2 {
    top: 8px;
    left: 96px;
}
.start-item-1_3 {
    top: 8px;
    left: 184px;
}
.start-item-1_4 {
    top: 8px;
    left: 272px;
}
.start-item-2_1 {
    top: 96px;
    left: 8px;
}
.start-item-2_2 {
    top: 96px;
    left: 96px;
}
.start-item-2_3 {
    top: 96px;
    left: 184px;
}
.start-item-2_4 {
    top: 96px;
    left: 272px;
}
.start-item-3_1 {
    top: 184px;
    left: 8px;
}
.start-item-3_2 {
    top: 184px;
    left: 96px;
}
.start-item-3_3 {
    top: 184px;
    left: 184px;
}
.start-item-3_4 {
    top: 184px;
    left: 272px;
}

.start-item-4_1 {
    top: 272px;
    left: 8px;
}
.start-item-4_2 {
    top: 272px;
    left: 96px;
}
.start-item-4_3 {
    top: 272px;
    left: 184px;
}
.start-item-4_4 {
    top: 272px;
    left: 272px;
}

.game-item_1 {
    background: #efe5da;
}
.game-item_2 {
    background: #ece0c6;
}
.game-item_4 {
    background: #efb27b;
}
.game-item_8 {
    background: #f77c5f;
}
.game-item_16 {
    background: #f55d3c;
}
.game-item_32 {
    background: #eece71;
}
.game-item_64 {
    background: #f77c5f;
}
.game-item_128 {
    background: #f2370b;
}
.game-item_256 {
    background: #9cabff;
}
.game-item_512 {
    background: #6d65ed;
}
.game-item_1024 {
    background: #3d32ec;
}
.game-item_2028 {
    background: #d5ec32;
}
</style>