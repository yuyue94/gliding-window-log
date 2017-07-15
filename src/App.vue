<template>
    <div id="app">
        <div id="title">
            <h2>Cooltag's Gliding Window Log Div</h2>
            <button v-if="isRunning" @click="stop">stop</button>
            <button v-else @click="start">start</button>
        </div>
        <gliding-window-log id="log" :cache="cache" :status="status" :capacity="50"></gliding-window-log>    
    </div>
</template>

<script>
import GlidingWindowLog from './GlidingWindowLog.vue'
export default {
    name: 'app',
    data() {
        return {
            cache: [],
            status: 'idle'
        }
    },
    computed: {
        isRunning() {
            return this.status === 'running';
        }
    },
    mounted() {

    },
    methods: {
        getRandomItem(list) {
            return list[this.getRandomInt(list.length)];
        },
        getRandomInt(range) {
            return Math.floor(Math.random()*range);
        },
        stop() {
            this.status = 'idle';
        },
        start() {
            let when = ['上午10点', '凌晨两点', '太阳刚出来的时候', '香港回归20周年之日', '世界末日当天'];
            let who = ['jade', 'alex', 'cooltag', 'rino', 'david', '范总'];
            let where = ['池塘边', '女装店', '马桶盖上', '冉冉升起的国旗下', '电脑边'];
            let what = ['吃饭', '睡觉', '做着不可描述的事情', '冲Q币', '假装跌倒', '吟诗作画', '带上了光荣的红领巾', '跳芭蕾舞'];
            let result = ['被举报了', '有喜了', '睡着了', '被看上了', '穿越到了清朝做了格格']

            let getStr = function() {
                let str = '在' + this.getRandomItem(when) +
                    '，' + this.getRandomItem(who) + 
                    '在' + this.getRandomItem(where) + 
                    this.getRandomItem(what) +
                    '，结果' + this.getRandomItem(result);
                return str;
            }.bind(this);

            let gotcha = true;
            let fakeGetData = function() {
                let logs = [];

                if (gotcha) {
                    let count = this.getRandomInt(100);
                    for (let i=0;i<count;i++) {
                        logs.push(getStr());
                    }
                }

                [].push.apply(this.cache, logs);
                if (this.isRunning) {
                    setTimeout(fakeGetData, 1000);
                }

                if (gotcha) {
                    gotcha = this.getRandomInt(100) > 40;
                } else {
                    gotcha = this.getRandomInt(100) > 80;
                }
            }.bind(this);

            this.status = 'running';
            fakeGetData();
        }
    },
    components: {
        'gliding-window-log': GlidingWindowLog
    }
}
</script>

<style>
html, body, #app {
    height: 100%;
    margin: 0;
}
body {
    padding: 30px 48px 0;
    box-sizing: border-box;
}
#title {
    height: 50px;
}
#title h2{
    float: left;
    margin: 0;
}
#title::after {
    clear: both;
    content: '';
    display: block;
}
#title button {
    float: right;
    font-size: 20px;
    cursor: pointer;
    border-radius: 4px;
    background-color: white;
    border: 1px solid #999;
    padding: 5px 20px;
}
#title button:hover {
    background-color: #ddd;
}
#title button:focus {
    outline: none;
}
</style>
