<template>
    <div class="index">
       <van-pull-refresh v-model="isLoading" @refresh="onRefresh">
            <van-list :class="[loading ? '' : 'van-pad']" v-model="loading" :finished="finished" finished-text="没有更多了" @load="getlist">
                <div class="grid" :style="{height: box_wrapper_h + 'px', paddingTop: gutter + 'px'}">
                    <div class="grid-item" 
                    :style="{width: item_w + 'px', height: item.height + 'px', transform: 'translate('+item.left +'px,'+ item.top +'px)'}"
                    v-for="(item, index) in list" :key="index" 
                    :data-top="item.top" :data-scrollTop="scrollTop"
                    v-if=" (item.top >= scrollTop - window_h) && (item.top <= scrollTop + window_h * 2)">
                        <img :src="item.src">
                    </div>
                </div>
            </van-list>
        </van-pull-refresh>
    </div>
</template>

<script>
    var img_arr = [
        require('@/assets/demo (1).jpg'),
        require('@/assets/demo (2).jpg'),
        require('@/assets/demo (3).jpg'),
        require('@/assets/demo (4).jpg'),
        require('@/assets/demo (5).jpg'),
        require('@/assets/demo (6).jpg'),
        require('@/assets/demo (7).jpg'),
        require('@/assets/demo (8).jpg'),
        require('@/assets/demo (9).jpg'),
        require('@/assets/demo (10).jpg'),
        require('@/assets/demo (11).jpg'),
        require('@/assets/demo (12).jpg'),
        require('@/assets/demo (13).jpg'),
        require('@/assets/demo (14).jpg'),
        require('@/assets/demo (15).jpg')
    ]
    export default {
        name: 'index',
        data () {
            return {
                list: [],
                loading: false,
                finished: false,
                isLoading: false,

                gutter: 10, // 组件间隔大小
                window_w: 375,
                window_h: 667,
                column: 2, // 展示列数
                item_w: 172.5,
                grid_h: [],
                box_wrapper_h: 0,

                scrollTop: 0,
                limit_hidden_page_num: 2
            }
        },
        mounted () {
            this.init()
            var self = this
            window.onscroll = function () {
                self.scrollTop = window.scrollY
            }
        },
        methods: {
            init () {
                this.window_w = window.innerWidth
                this.window_h = window.innerHeight
                this.item_w = (this.window_w - ((this.column + 1) * this.gutter)) / this.column
                for (var i = 0; i < this.column; i++) {
                    this.grid_h[i] = 0
                }
            },
            getlist (refresh) {
                var arr = []
                for (var i = 0; i < 10; i++) {
                    var index = parseInt(Math.random() * 15)
                    arr.push({
                        src: img_arr[index]
                    })
                }
                setTimeout(() => {
                    this.loading = false
                    this.isLoading = false
                    this.calculation(arr).then(res => {
                        this.list = this.list.concat(res)
                        if (refresh && this.box_wrapper_h < window.innerHeight) {
                            this.getlist('refresh')
                        }
                    })
                    if (this.list.length >= 10000) {
                        this.finished = true
                    }
                }, 1000)
            },
            calculation (arr) {
                return new Promise((resolve, reject) => {
                    var self = this
                    var _arr = []
                    var reduce_arr = function (img_arr) {
                        var item = img_arr.splice(0, 1)[0]
                        var img = new Image()
                        img.src = item.src
                        img.onload = function () {
                            var max_index = self.max_item()
                            item.height = self.item_w * img.height / img.width
                            item.top = self.grid_h[max_index] == 0 ? 0 : self.grid_h[max_index]
                            item.left =  max_index * (self.item_w + self.gutter) + self.gutter
                            self.grid_h[max_index] = self.grid_h[max_index] + item.height + self.gutter
                            _arr.push(item)
                            self.box_wrapper_h = self.grid_h[max_index]
                            if (img_arr.length > 0) {
                                reduce_arr(img_arr)
                            } else {
                                resolve(_arr)
                            }
                        }
                    }
                    reduce_arr(arr)
                })
            },
            max_item () {
                var max = Math.min(...this.grid_h)
                return this.grid_h.indexOf(max)
            },
            onRefresh () {
                this.list = []
                this.finished = false
                this.isLoading = true
                this.init()
                this.getlist('refresh')
            }
        }
    }
</script>

<style scoped>
    .index {
        font-size: 32px;
    }
    .grid-item img {
        width: 100%;
        display: block;
        border-radius: 8px;
    }
    .grid {
        position: relative;
    }
    .grid-item {
        position: absolute;
        background-color: #f8f8f8;
        border-radius: 8px;
    }
    .van-pad {
        padding-bottom: 50px;
    }
</style>