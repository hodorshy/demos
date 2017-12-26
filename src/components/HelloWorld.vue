<template>
    <div class="hello" ref="app">
        <!-- <h1>{{ msg }}</h1>
        <h2>Essential Links</h2>
        <img :src="imgUrl" alt="url error"> -->


        <!-- test -->
        <!-- <Test v-if="msg"></Test> -->

        <!-- swiper -->
        <!-- <swiper></swiper> -->
        <!-- <WcSwiper>
            <div slot="index11">
                <h1>xaxxxxxaslkjfas;lfjlj</h1>
            </div>
        </WcSwiper> -->


        <!-- bottomMenu -->
        <h1 @click="test">res: {{ bottomList.switch }} {{ index1 }}</h1>
        <ButtomMenu id="bottomMenu"
                    :opt="bottomList"
                    :words="words"
                    @bottomCallback="bottomCallback">
            <div slot="item_2" slot-scope="props">
                <BottomItem
                    :item="props.item"

                    :index="props.index"
                    :curIndex.sync="bottomList.curIndex">

                    <span slot="word">{{ index5 }}</span>
                </BottomItem>
            </div>
        </ButtomMenu>
    </div>
</template>

<script>

// import Test from './test.vue'
import swiper from './vue-swipe.vue'
import WcSwiper from './wcSwiper.vue'
import ButtomMenu from './BottomList/BottomMenu.vue'
import BottomItem from './BottomList/BottomItem.vue'
import Vue from 'Vue'


let Dictionary = {
    index1: [ '开启', '关闭' ],
    index2: [ 'A模式', 'B模式', 'C模式', 'D模式' ],
    index3: [ '测试1', '测试2', '测试3', '测试4' ],
    index4: [ '测试1', '测试2', '测试3', '测试4' ],
    index5: [ '测试1', '测试2', '测试3', '测试4' ],
}

Vue.filter( 'switch', $index => Dictionary.index1[ $index ] );

export default {
    name: 'HelloWorld',
    data() {
        return {

            Bswitch: false,
            index1: 1,
            index2: 2,
            index3: 1,
            index4: 0,
            index5: 'eee',

            itemCustom: {
                arrow: false,
                imgUrl: ['/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg'],
                activeCallback: (index, item) => { console.log( 1 ) },
                ungroup: true,
            },

            bottomList: {
                data: [
                    { arrow: false, imgUrl: [ '/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg' ], callback: index => console.log( 'bottomItemCallback:', index ) },
                    { arrow: true, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
                    // { arrow: false, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
                    {
                        arrow: true,
                        imgUrl: ['/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg'],
                        activeCallback: (index, item) => { console.log( 1 ) },
                        callback: ( index, item ) => {
                            // item.isActive = !item.isActive;
                            console.log('item', item);
                        }
                    },
                    { ungroup: true, arrow: false, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
                 ],
                arrowUrl: '/src/components/BottomList/images/off.jpg',
                curIndex: -99,
                switchIndex: 0,
                switch: false,
                childSwitchIndex: 3,
                childSwitch: false,
            },

        }
    },

    created() {
        // this.bottomList.data[ 2 ].callback = ( index, item ) => {
        //     // this.bottomList.curIndex = index;
        //     // item.isActive = !item.isActive;
        //     console.log( 'ungoup', item );
        // }
    },

    mounted() {

    },

    components: {
        // Test,
        // 'Test': () => import('./test.vue'),
        swiper,
        WcSwiper,
        ButtomMenu,
        BottomItem,
    },

    computed: {
        words() {
            return [ Vue.filter( 'switch' )( +this.bottomList.switch ), this.index2, this.index3, this.index4 ];
        },
        switch() {
            return this.bottomList.switch;
        }
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

        test() {
            console.log( 'click' );
            this.index1 = +( !this.index1 );
            this.bottomList.curIndex = 3;
            // this.bottomList.words[ 0 ] = '123';
        },

        bottomCallback( $index ) {

            switch ( $index ) {
                case this.bottomList.switchIndex:
                    // this.bottomList.data[ 2 ].isActive = false;
                    break;

                default:
                    break;
            }
        }

    },

    filters: {
        'switch'( $index ) {
            return Dictionary.index1[ $index ];
        }
    }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
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

body {
    padding: 0;
    margin: 0;
}

#bottomMenu {
    height: 140px;
    width: 100%;
    overflow: hidden;
}
</style>
