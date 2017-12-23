<template>
    <div class="hello" ref="app">
        <!-- <h1>{{ msg }}</h1>
        <h2>Essential Links</h2>
        <img :src="imgUrl" alt="url error"> -->


        <!-- test -->
        <!-- <Test v-if="msg"></Test> -->

        <!-- swiper -->
        <!-- <swiper></swiper> -->
        <WcSwiper>
            <div slot="index11">
                <h1>xaxxxxxaslkjfas;lfjlj</h1>
            </div>
        </WcSwiper>
    </div>
</template>

<script>

// import Test from './test.vue'
import swiper from './vue-swipe.vue'
import WcSwiper from './wcSwiper.vue'

export default {
    name: 'HelloWorld',
    data() {
        return {
            msg: 'Welcome to Your Vue.js App',
            imgUrl: '../../static/images/2.jpg'
        }
    },

    mounted() {

    },

    components: {
        // Test,
        // 'Test': () => import('./test.vue'),
        swiper,
        WcSwiper,
    },

    methods: {

        /* 函数防抖 */
        debounce(func, wait, immediate) {
            var timeout, args, context, timestamp, result;
            var now = new Date();

            var later = function() {
                var last = now - timestamp;

                if (last < wait && last >= 0) {
                    timeout = setTimeout(later, wait - last);
                } else {
                    timeout = null;
                    if (!immediate) {
                        result = func.apply(context, args);
                        if (!timeout) context = args = null;
                    }
                }
            };

            return function() {
                context = this;
                args = arguments;
                timestamp = new Date();
                var callNow = immediate && !timeout;
                if (!timeout) timeout = setTimeout(later, wait);
                if (callNow) {
                    result = func.apply(context, args);
                    context = args = null;
                }

                return result;
            };
        },

    },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1,
h2 {
    font-weight: normal;
}

ul {
    list-style-type: none;
    padding: 0;
}

li {
    display: inline-block;
    margin: 0 10px;
}

a {
    color: #42b983;
}
</style>
