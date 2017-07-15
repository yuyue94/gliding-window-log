<template>
    <div :id="id" class="gliding-window-log">
        <div v-for="item in logWindow" :id="'log-line-'+item.index" :key="item.index">
            <span class="line-index">{{item.index}}</span>
            <div v-if="item.line" class="line-content">
                {{item.line}}
            </div>
            <br v-else/>
        </div>
    </div>
</template>

<script>
export default {
    props: ['id', 'cache', 'status', 'capacity'],
    data() {
        return {
            tick: 1, // 窗口数
            currentEndIndex: -1,  // 记录改变lockScroll状态时的最末log的index
            lockScroll: true, // 滚动锁定，为true时始终滚动条沉底
            output: []
        }
    },
    computed: {
        logWindow() { 
            // 当前显示的log窗口的所有log
            if (this.lockScroll) { };  
            // 只是告诉在lockScroll更新时刷新logWindow
            // 因为有暗箱操作，所以需要在滚动条触底时（lockScroll为true）更新logWindow
            let size = this.capacity * this.tick;
            let start = this.currentEndIndex - size + 1;
            if (start < 0) {
                start = 0;
            }
            return this.output.slice(start, this.currentEndIndex + 1);
        },
        isRunning() {
            return this.status === 'running';
        }
    },
    watch: {
        status() {
            if (this.isRunning) {
                this.output = [];
                this.addLine();
            }
            this.tick = 1;
            this.lockScroll = true;
        },
        output() {
            if (this.lockScroll) {
                this.$nextTick(() => {
                    this.scrollToBottom();
                });
            }
            this.currentEndIndex = this.output.length - 1;
        },
        lockScroll() {
            if (this.isRunning) {
                this.tick = 1;
            }
            this.currentEndIndex = this.output.length - 1;
        }
    },
    methods: {
        scrollToBottom() {
            let logBody = document.getElementById(this.id);
            if (logBody) {
                logBody.scrollTop = logBody.scrollHeight;
            }
        },
        bindScrollbar() {
            let logDiv = document.getElementById(this.id);
            logDiv.addEventListener('scroll', e => {
                if (this.output.length > this.capacity) {
                    if (e.target.scrollTop + logDiv.clientHeight === logDiv.scrollHeight) { 
                        // 判断触底 
                        this.lockScroll = true;
                    } else {
                        // 尝试往上滚
                        this.lockScroll = false;
                    }
                }

                if (
                    ((this.isRunning && !this.lockScroll) || !this.isRunning)
                    && this.output.length !== 0
                    && e.target.scrollTop === 0
                ) { // 触顶增加tick
                    let last = this.logWindow[this.logWindow.length - 1];
                    let index = last.index - this.capacity * this.tick;
                    if (index > 0 && this.logWindow.length < this.output.length) {
                        this.tick++;
                        this.$nextTick(() => {
                            document.getElementById('log-line-' + index).scrollIntoView();
                        });
                    }
                }
            });
        },
        // cache向output推num条日志，silent决定是否触发output的数组侦听
        flush(num, silent) {
            silent = silent || false;
            let intercept = this.cache.splice(0, num).map((item, i) => {
                return {
                    line: item,
                    index: this.output.length + i
                }
            });

            if (silent) {
                // vue变异方法是在继承了原生方法的基础上写的监听数组的方法，所以用原生的push.apply不会有监听
                [].push.apply(this.output, intercept);
            } else {
                this.output = this.output.concat(intercept);
            }
        },
        // 增加行数，考虑不同状态下的向output推log的策略
        // 1. 在运行且非滚动锁定时，silent为true，界面logWindow不会因为output变化而变化
        // 2. 没有在运行时（完成，停止，失败），日志直接全给，silent为false
        // 3. 在运行时且滚动锁定时，50条以外的log直接给，最新的50条一条条刷动画
        addLine() {
            if (this.isRunning && !this.lockScroll || !this.isRunning) {
                // 为了避免运行＋滚动的性能问题，就不实时给界面了
                this.flush(this.cache.length, this.isRunning);
            } else {
                if (this.cache.length > 50) {  //  只刷最后50条
                    this.flush(this.cache.length - 50);
                } else {
                    let item = this.cache.shift();
                    if (item) {
                        this.output.push({
                            line: item,
                            index: this.output.length
                        });
                    }
                }
            }
            if (this.isRunning || this.cache.length > 0) {
                let frequency = Math.min(1 / this.cache.length * 1000, 100);
                // let frequency = 1000;
                setTimeout(this.addLine.bind(this), frequency);
            }
        }
    },
    mounted() {
        this.bindScrollbar();
    }
}
</script>

<style scoped>
.gliding-window-log {
    overflow: auto;
    height: calc(100% - 98px);
    margin-bottom: 48px;
    background-color: #37393D;
    box-sizing: border-box;
    position: relative;
    font-size: 16px;
    line-height: 22px;
}
.gliding-window-log .line-index {
    position: absolute;
    left: 0;
    color: #4D4E50;
    width: 40px;
    background-color: #2E2E2E;
    text-align: right;
    padding-right: 10px;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
}
.gliding-window-log .line-content {
    white-space: nowrap;
    margin-left: 40px;
    padding-left: 20px;
    color: #ccc;
}
.gliding-window-log .line-content:hover {
    background-color: #555;
}
</style>
