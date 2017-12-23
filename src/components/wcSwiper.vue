<template>
    <div>
        <wc-swiper>
            <wc-slide>
                <slot :name="name + 11"></slot>
            </wc-slide>
            <wc-slide>
                <h1>22222</h1>
            </wc-slide>
        </wc-swiper>
    </div>
</template>

<script>
import Vue from 'vue'
import wcSwiper from 'wc-swiper'
import 'wc-swiper/style.css'
Vue.use(wcSwiper);

export default {
    data() {
        return {
            name: 'index'
        }
    }
}
</script>

<style scoped lange="less">
.swiper {
    height: 200px;
}
</style>
