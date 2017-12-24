<template>
    <div>

        <!-- 按钮小于 4 个 -->
        <div class="bottom-simple" ref="bottomSimple" v-if=" length <= COUNT_MAX ">
            <div :ref=" 'item_' + $index " v-for=" ( item, $index ) in data " :key="$index"
                :class="[ 'item', 'item_' + $index ]"
                @click=" tap( $index ) ">
                <slot :name=" 'item_' + $index " :isSwitch=" opt.switchIndex === $index ">
                    <BottomItem
                        :class="[ { switch: opt.switchIndex === $index }, { child: opt.childSwitch === $index }, { ungroup: item.ungroup } ]"
                        :word="words[ $index ]"
                        :index="$index + 1"
                        :arrow=" item.arrow && opt.arrowUrl "
                        :curIndex.sync="opt.curIndex"
                        :isSwitch=" opt.switchIndex === $index "
                        :isChildSwitch=" opt.childSwitchIndex === $index "
                        :item="item"
                        @childMode="childMode" >

                    </BottomItem>
                </slot>
            </div>
        </div>

        <!-- 按钮超过 4 个 -->
        <div class="bottom-swiper" v-else>
            <h1>this is swiper</h1>
        </div>

    </div>
</template>

<script>

/*  引用方式:

    <ButtomMenu :opt="bottomList"
                :words="words"
                @bottomCallback="bottomCallback">
        <!-- <div slot="item_2">xxx</div> -->
    </ButtomMenu>
*/

/*  传入的数据格式:

    bottomList: {
        data: [
            { ungroup: false, arrow: false, imgUrl: [ '/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg' ], callback: index => console.log( 'bottomItemCallback:', index ) },
            { ungroup: false, arrow: true, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
            { ungroup: false, arrow: false, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
            { ungroup: true, arrow: false, imgUrl: '/src/components/BottomList/images/on.jpg', callback: index => console.log( 'bottomItemCallback:', index ) },
            ],
        arrowUrl: '/src/components/BottomList/images/off.jpg',
        curIndex: -99,
        switchIndex: 0,
        switch: false,
        childSwitchIndex: 3,
        childSwitch: false,
    },

    words() {
        return [ this.index1, this.index2, this.index3, this.index4 ];
    },
*/

/**
 * 属性:
 * ungroup: ( 可选: 默认 false )    代表是否同组互斥关系, 为 true 不同组, 可以单独控制点击状态, 为 false 同组, 有互斥关系;
 * imgUrl: ( 必须 )                 图片路径, 如果为数组, 背景图片可以替换( 0, 1 之间替换 )
 * callback: ( 可选 )               回调函数, 每个单独的回调函数, 也可以写在 bottomCallback 事件里, 通过 index 监听( 效果一样 );
 * bottomCallback ( 可选 )          点击事件
 *
 *
 *  自定义, 里面可以是其他组件
    <div slot="item_2">
        <BottomItem
            :word="111111111111"
            :index="3"
            :curIndex.sync="bottomList.curIndex"
            :item="{ arrow: false, imgUrl: ['/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg'] }" >
        </BottomItem>
    </div>
 *
 *
 *
 * 注意点:  ================================ !!!!!!!!!!!!!!!!!!!!!!!
 *  1. curIndex 是从 1 开始递增的;
 *  2. 如果有特殊的童锁, 不要传入 childSwitch 和 childSwitchIndex;
 *  3. 童锁关闭, 其他按钮默认保留上次状态;
 *  4. 开关关闭再开始默认保留上次状态;
 *  5. words 数组控制显示内容, 使用计算属性;
 *  6. 特殊需求, 可以使用 <div slot="item_1">xxx<div> 来自定义
 *
 */

import BottomItem from './BottomItem.vue'

export default {
    name: 'bottom-menu',
    props: [ 'opt', 'words' ],

    components: {
        BottomItem,
    },

    data() {
        return {
            COUNT_MAX: 4,
        };
    },

    computed: {
        data() {
            return this.opt.data;
        },
        length() {
            return this.data.length;
        },
    },

    mounted() {

        // 设置宽度
        if ( this.length <= this.COUNT_MAX ) {
            this.$refs.bottomSimple.querySelectorAll( '.item' ).forEach( el => el.style.width = 100 / this.length + '%' );
        }
    },

    watch: {
    },

    methods: {
        /* 点击事件 */
        tap( $index ) {

            switch ( $index ) {

                case this.opt.switchIndex:              // 开关
                    this.clickSwitch( this.opt );
                    break;

                case this.opt.childSwitchIndex:         // 童锁
                    this.opt.childSwitch = !this.opt.childSwitch;
                    break;
            }

            // 开关管 || 为 disable 状态
            if ( !this.opt.switch || this.$refs[ 'item_' + $index ][ 0 ].querySelector( '.icon' ).classList.contains( 'disable' ) ) return;

            this.$emit( 'bottomCallback', $index );
        },

        /* 开关点击事件 */
        clickSwitch( opt ) {

            opt.switch = !opt.switch;
            opt.curIndex = opt.switch ? Math.abs( opt.curIndex ) : -opt.curIndex;
            if ( opt.switch && opt.childSwitchIndex >= 0 && opt.childSwitchIndex < this.length ) {
                this.childMode( opt.data[ opt.childSwitchIndex ].isActive = false );
            }
        },

        /* 童锁控制 disable 状态 */
        childMode( isActive ) {
            this.opt.data.forEach( ( el, index ) => index !== this.opt.switchIndex && index !== this.opt.childSwitchIndex && ( el.isDisable = isActive ) );
        },
    },
}
</script>

<style scoped lang="less">

.setHW() {
    width: 100%;
    height: 100%;
}

.bottom-simple {
    .setHW();
    background-color: red;

    .item {
        height: 100%;
        background-color: green;
        float: left;
    }
}

</style>

