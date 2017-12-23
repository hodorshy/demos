<template>
    <div class="swiper"  @click="test">
        <swipe ref="swiper" class="my-swipe" :auto="0" :defaultIndex="defaultIndex">
            <swipe-item class="slide1"></swipe-item>
            <swipe-item class="slide2"></swipe-item>
            <swipe-item class="slide3"></swipe-item>
        </swipe>
    </div>
</template>

<script>

// https://github.com/ElemeFE/vue-swipe

import { Swipe, SwipeItem } from 'vue-swipe';
// import rdSwipe from 'vue-slide'
import 'vue-swipe/dist/vue-swipe.css'

export default {

    props: [ 'auto' ],

    components: {
        Swipe,
        SwipeItem,
    },

    data() {
        return {
            defaultIndex: 1,
        }
    },

    created() {
        // this.auto = 200;
    },

    methods: {
        test() {
            console.log( 'click', this.$refs.swiper );
            this.defaultIndex = 2 ;
        },
    },
}

</script>

<style scoped lang="less">

.my-swipe {
  height: 200px;
  color: #fff;
  font-size: 30px;
  text-align: center;
}

.slide1 {
  background-color: #0089dc;
  color: #fff;
}

.slide2 {
  background-color: #ffd705;
  color: #000;
}

.slide3 {
  background-color: #ff2d4b;
  color: #fff;
}

</style>

