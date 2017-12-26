<template>
    <div ref="item">

        <!-- 图标 -->
        <slot name="icon">
            <div ref="icon"
                :class="[ 'icon', { active: isActive }, { disable: isDisable } ]"
                :style="{ backgroundImage: 'url(' + imgUrl + ')' }"
                @click=" tap " ></div>
        </slot>

        <!-- 底部文字 -->
        <p class="word">
            <slot name="word" :word="word">{{ word }}</slot>
            <span :class="{ arrow: arrow }" :style="{ backgroundImage: 'url(' + arrow + ')' }"></span>
        </p>

    </div>
</template>

<script>

/*
    引用方式:
    <BottomItem
        :word="words[ $index ]"
        :index="$index + 1"
        :arrow=" item.arrow && opt.arrowUrl "
        :curIndex.sync="opt.curIndex"
        :isSwitch=" opt.switchIndex === $index "
        :isChildSwitch=" opt.childSwitchIndex === $index "
        :item="item" >
    </BottomItem>
 */

/*
    item: {
        arrow: false,
        imgUrl: [ '/src/components/BottomList/images/on.jpg', '/src/components/BottomList/images/off.jpg' ],
        callback: index => console.log( 'bottomItemCallback:', index ),

        // 这里补充的
        isActive: false,    // 控制是否会活动状态
        isDisable: false,   // 控制是否为不可点击状态
    }
 */

/**
 *
 * 传入的数据:
 *  item: arrow, imgUrl, callback 详情看 BottomMenu.vue
 *
 */



const SWITCH  = 'switch';
const CHILD   = 'child';
const UNGROUP = 'ungroup';

export default {
    name: 'bottomItem',
    props: [ 'word', 'isSwitch', 'isChildSwitch', 'index', 'arrow', 'curIndex', 'item' ],

    data() {
        return {
            imgIndex: 0,
            unClick: false,
            isToggleImg: false,
        };
    },

    created() {

        // 当为 ungroup 时, 设置控制 active 的 isActive 值
        this.$set( this.item, 'isActive', false );
        // 默认状态为 disable
        this.$set( this.item, 'isDisable', false );
        // 是否切换图片
        this.isToggleImg = Array.isArray( this.item.imgUrl );
        this.isCustom = this.curIndex == null;
    },

    computed: {
        imgUrl() {
            return !this.isToggleImg ? this.item.imgUrl : this.item.imgUrl[ this.imgIndex ];
        },

        /* 控制是否会活动状态, 计算属性, 不能设置 */
        isActive() {

            // 是否保留童锁之前的状态
            console.log( !this.isSwitch , !this.item.ungroup , this.curIndex == this.index );
            return /* !this.isDisable &&  */( !this.isSwitch && !this.item.ungroup && this.curIndex == this.index ) || ( this.item.isActive && ( this.curIndex > 0 || this.isCustom ) );
        },

        /* 控制是否为不可点击状态, 计算属性, 不能设置 */
        isDisable() {
            return !this.isSwitch && this.curIndex < 0 || this.item.isDisable;
        },
    },

    methods: {
        tap() {

            // 开关管就返回
            if ( this.isDisable ) return;

            // 不是组内元素
            if ( this.item.ungroup && !this.isCustom ) this.item.isActive = !this.item.isActive;

            // 更新 changeCurIndex
            if ( !this.item.ungroup && !this.isCustom ) this.$emit( 'update:curIndex', this.index );
            // 单独触发 callback
            typeof this.item.callback === 'function' && this.item.callback( this.index, this.item );
        },
    },

    watch: {
        isActive() {
            typeof this.item.activeCallback === 'function' && this.item.activeCallback( this.index, this.item );
            // 童锁
            if ( this.isChildSwitch )  this.$emit( 'childMode', this.isActive );
        },
        word() {
            // 切换背景图
            if ( this.isToggleImg ) this.imgIndex = +( !this.imgIndex );
        },
    },

    mounted() {
        // 添加自身
        this.item.ref = this.$refs.item;

    }

}
</script>

<style lang="less">

@icon_width: 64px;
@arrow_width: 20px;
@word_height: 20px;

.setBgi() {
    background-repeat: no-repeat;
    background-size: cover;
    background-position: center;
}

.icon {
    width: @icon_width;
    height: @icon_width;
    border-radius: 50%;
    margin: auto;
    .setBgi();
}

.icon.active {
    background-color: black;
}

.icon.disable {
    opacity: .3;
}

.word {
    height: @word_height;
    text-align: center;
}

.word .arrow  {
    display: block;
    float: right;
    width: @arrow_width;
    height: 100%;
    .setBgi();
}

</style>