<template>
    <div class="colorPicker">
        <div class="picker" ref="picker">

            <canvas class="canv" ref="canv"></canvas>

            <!-- 遮罩图片 -->
            <img class="img" :src="imgUrl" ref="img" alt="路径错误">

            <!-- 遮罩层 -->
            <div class="mask center" ref="mask">

                <!-- 向外提供接口 -->
                <slot name="mask"></slot>
            </div>

            <!-- 拖动的圆 -->
            <div class="circle" ref="circle" :style="{ backgroundColor: color }"></div>
        </div>
    </div>
</template>

<script>

/**
 *
 * <Color-picker :width="width" :color.sync="color" :img-url="imgUrl" :is-render.sync:="isRender" :init-color="initColor" :callback="callback"></Color-picker>
 *
 * 注意: 父组件应该传入 width color maskWidth;
 *  width:      圆环外围半径                           (必须)
 *  color:      存储颜色值                             (必须)
 *  imgURL:     圆环图片地址地址                        (必须)
 *  isRender:   是否正在拖动                           (可选)
 *  initColor:  设备上报的初始颜色值                    (可变)
 *  callback:   触摸结束并且颜色值发生改变时触发的时间    (可选)
 *
 * 图片原始尺寸: 1104 x 1104
 */


// 默认配置
const def = {
    width     : 1104,                       // 初始宽高
    height    : 1104,
    lineWidth : 56 + 12 + 12,               // 圆环宽度
    Rmin      : 1104 / 2 - 56 - 12 - 12,    // 圆环最小半径
    circleW   : 56 + 12 + 12,               // 拖动圆宽度
    radius    : 1104 / 2 - 12 - 56 / 2,     // 圆环中心半径
    OFFSET    : 30,                         // 颜色值的偏差范围
    stepDeg   : 360,                         // 角度步长
    maskWidth : 70,                         // 遮罩层宽高
    initColor : 'rgba(100, 134, 251, 1)',
    normalW   : 26,
    bigW      : 34,
};

const reg = /(\d+),\s*(\d+),\s*(\d+),\s*(\d+)/;

let x = Symbol('x'), y = Symbol('y'), flag = false;

let cache = {};

export default {

    props: [ 'width', 'color', 'imgUrl', 'isRender', 'initColor', 'callback' ],

    data() {
        return {
            count    : 0,                   // 内部计数用
            colors   : [],                  // 临时存储用
            [x]      : undefined,           // 盒子内部的 x 坐标
            [y]      : undefined,           // 盒子内部的 y 坐标
            queue    : undefined,           // 图片加载的 promise 实例
            timer    : null,                // 定时器的 ID
            padding  : undefined,           // picker 的 padding 值
            target   : null,                // 存储目标元素
            isAction : true,                // 是否执行 action
            OFFSET   : 20,                  // 颜色值的偏差值
            cw       : 0,
        }
    },

    /* 初始化属性 */
    created() {

        // 如果没有传入 width 取默认值
        if ( this.width  == null ) this.width = def.width;
        // 如果没有传入 maskWidth 取默认值
        // if ( this.maskWidth == null ) this.maskWidth = def.width;

        this.scale      = this.width / def.width;           // 图片缩放比例
        this.lineWidth  = this.scale * def.lineWidth;       // 圆环宽度 (包括透明)
        this.Rmax       = this.width / 2;                   // 圆环半径 大 (包括透明)
        this.Rmin       = def.Rmin * this.scale;            // 圆环半径 小 (包括透明)
        this.circleW    = def.circleW * this.scale;         // 拖动的圆的宽度
        // this.circleW    = def.normalW;
        this.circleR    = this.circleW / 2;                 // 拖动的圆的半径
        this.radius     = def.radius * this.scale;          // 圆环中点半径
        this.cw         = def.normalW - this.circleW;
    },

    mounted() {

        this.padding    = this.getStyle( this.$refs.picker, 'padding' );

        let img         = this.$refs.img;
        this.queue      = new Promise( function (resolve, reject) {
            img.onload  = resolve;
            img.onerror = reject;
        } );

        img.src         = this.imgUrl;

        let
            width       = this.width,
            height      = this.width,
            canv        = this.$refs.canv;

        this.ctx        = canv.getContext( '2d' );

        // 设置样式 (宽高, 定位等尺寸)
        this.setStyle( img, { width, height } );
        this.setStyle( canv, { width, height } );
        this.setStyle( this.$refs.picker, { width, height } );

        // 遮罩层样式 (宽高)
        this.setStyle( this.$refs.mask, {
            width  : this.radius * 2 - def.maskWidth - this.lineWidth,
            height : this.radius * 2 - def.maskWidth - this.lineWidth,
        } );

        // normal Circle
        this.norMalCircle();

        // 设置 canvas 宽高
        Object.assign( canv, { width, height } );

        // 图片 onload 时间
        this.queue = this.queue.then( _ => this.ctx.drawImage( img, 0, 0, width, height ) );

        // 设置初始位置
        this.setCircleTopLeft( { x: this.Rmax - this.circleR, y: 0 } );
        // 上报颜色值初始化
        this.getInitPoint( this.initColor );

        // 注册事件
        this.$refs.picker.addEventListener( 'touchmove', this.moveCircle );
        this.$refs.picker.addEventListener( 'touchend', this.moveEnd );
        this.$refs.picker.addEventListener( 'touchstart', this.moveStart );
    },

    methods: {
        /**
         * touchmove
         */
        moveCircle(e) {

            let target  = e.target;
            if ( e.targetTouches.length > 1 || target === this.$refs.mask ) return;
            !this.isRender && this.$emit('update:isRender', true);

            if ( target === this.$refs.circle ) target = this.$refs.canv;

            let point   = e.targetTouches[0];
            this[x]     = point.pageX - this.getLeft( target );
            this[y]     = point.pageY - this.getTop( target );

            this.target = target;
            if (this.isAction) this.action();

            e.preventDefault();
        },
        /**
         * touchend
         */
        moveEnd(e) {

            if ( this.isRender ) {
                this.norMalCircle();
                // 解决变小的时候产生的偏移量
                this.setStyle( this.$refs.circle, {
                    top  : this.getStyle( this.$refs.circle, 'top' ) + ( def.bigW - def.normalW ) / 2,
                    left : this.getStyle( this.$refs.circle, 'left' ) + ( def.bigW - def.normalW ) / 2,
                } );
            }

            /* callback */
            if (  isRender ) typeof this.callback === 'function' && this.callback( this.color );

            this.$emit( 'update:isRender', false );

            window.cancelAnimationFrame( this.timer );
            // window.clearInterval(this.timer);
            this.isAction = true;
            e.preventDefault();
        },
        /**
         * touchstart
         */
        moveStart(e) {
            let target = e.target, point, res;

            if ( target === this.$refs.mask || target === this.$refs.circle ) return;
            this.cw = def.normalW - this.circleW;
            this.norMalCircle();

            // 解决变小的时候产生的偏移量
            this.setStyle( this.$refs.circle, {
                top  : this.getStyle( this.$refs.circle, 'top' ) + ( def.bigW - def.normalW ) / 2,
                left : this.getStyle( this.$refs.circle, 'left' ) + ( def.bigW - def.normalW ) / 2,
            } );

            point      = e.targetTouches[0];
            res        = this.getTargetPoint( point.pageX - this.getLeft(target), point.pageY - this.getTop(target), target );

            this.setCircleTopLeft( res );
            e.preventDefault();
        },

        /**
         * 拖动效果
         */
        action() {

            this.bigCircle();
            this.timer    = window.requestAnimationFrame( this.action.bind( this ) );
            this.isAction = false;
            this.setCircleTopLeft( this.getTargetPoint( this[x], this[y], this.target ) );

            // this.isAction = false;
            // this.timer = window.setInterval( _ => {

            //     this.setCircleTopLeft(this.getTargetPoint( this[x], this[y], this.target ));
            // }, 16.7);
        },

        /**
         * 计算坐标, 根据鼠标坐标获取对应圆环坐标
         * @param ( offsetX, offsetY ) 在 canvas 上的坐标
         * @param el 事件源, dom 对象
         * @returns { Object } { x, y } 对应在圆环上的坐标
         */
        getTargetPoint(offsetX, offsetY, el) {

            let target = {}, padding = ( el === this.$refs.picker ? this.padding : 0 ), x, y;

            let center = {
                x: this.getStyle( el, 'width' ) / 2 + padding,
                y: this.getStyle( el, 'height' ) / 2 + padding
            };

            x = offsetX - center.x;
            y = offsetY - center.y;

            if ( x !== 0 ) {

                let deg  = Math.atan( y / x);

                target.x = ( x > 0 ? Math.abs( this.radius * Math.cos( deg ) ) : -Math.abs( this.radius * Math.cos( deg ) ) ) + this.radius;
                target.y = ( y > 0 ? Math.abs( this.radius * Math.sin( deg ) ) : -Math.abs( this.radius * Math.sin( deg ) ) ) + this.radius;
            } else {    // 解决 90, 270 度的时候 没有斜率的问题

                target = { x: this.radius, y: this.radius + ( y > 0 ? this.radius : -this.radius ) }
            }

            return target;
        },

        /**
         * 根据设备上报颜色值, 初始化角度
         * @param color 颜色值, 一般是 this.initColor
         */
        getInitPoint(color) {
            console.time( 'initColor' );

            let error      = 'Don\'t have this initColor: ' + color;
            if ( color == null ) throw new Error( error );

            const colorArr = color.match( reg ).splice( 1, 4 );
            const deg      = Math.PI * 2;
            const step     = deg / def.stepDeg;

            if ( colorArr[0] < 68 - def.OFFSET || colorArr[1] < 85 - def.OFFSET || colorArr[2] < 112 - def.OFFSET ) throw new Error( error );

            let absArr = [], startDeg, endDeg, key, data;

            /* 获取区间 */
            startDeg = ( getPosition(colorArr) - 1 ) * Math.PI / 2;
            endDeg   = startDeg + Math.PI / 2;

            this.queue
                /* 遍历匹配 canvas 上的颜色值 */
                .then( _ => {
                    for ( let i = startDeg; i <= endDeg; i += step ) {
                        this.colors.push( [this.radius * Math.cos( i ), this.radius * Math.sin( i )] );

                        let x, y, res, a, b, c, el;

                        el = this.getColor( this.ctx.getImageData(
                                    x = this.colors[this.count][0] + this.Rmax,
                                    y = this.colors[this.count][1] + this.Rmax,
                                    1,
                                    1
                                ).data ).match( reg ).splice( 1, 3 );

                        a = Math.abs( el[0] - colorArr[0] );
                        b = Math.abs( el[1] - colorArr[1] );
                        c = Math.abs( el[2] - colorArr[2] );

                        if ( ( res = a + b + c ) < def.OFFSET ) absArr.push( [x, y] );
                        this.count ++;
                    }

                    if ( absArr.length === 0 ) throw new Error( error );

                    let point    = absArr[Math.floor(absArr.length / 2)];
                    let [x, y]   = [point[0] - this.circleR, point[1] - this.circleR];
                    this.setCircleTopLeft( { x, y } );

                    console.timeEnd( 'initColor' );
                }).catch( err => {

                    this.initError( err );
                    console.error( err.message );
                } );

            /**
             * 根据颜色值来获取分区
             * @returns { Number } 分区号
             */
            function getPosition(colorArr) {

                let a = colorArr[0], b = colorArr[1], position;

                if ( a <= 253 && a >= 167 && b >= 103 && b <= 130 ) {
                    position = 1;
                } else if ( a >= 68 && a <= 167 && b >= 102 && b <= 190 ) {
                    position = 2;
                } else if ( a >= 68 && a <= 179 && b >= 190 && b <= 226 ) {
                    position = 3;
                } else {
                    position = 4;
                }
                return position;
            }
        },

        /**
         * 色盘没有 initColor 的时候处理函数
         */
        initError(error) {
            // 给定一个默认值
            // this.getInitPoint(def.initColor);
        },

        /**
         * 提交 color 值
         */
        commitColor(data) {
            this.$emit( 'update:color' , data );
        },

        /**
         * 获取页面 top 值
         */
        getTop(el){
            let offset = el.offsetTop;
            if ( el.offsetParent != null ) offset += this.getTop( el.offsetParent );
            return offset;
        },
        /**
         * 获取页面 left 值
         */
        getLeft(el) {
            let offset = el.offsetLeft;
            if ( el.offsetParent != null ) offset += this.getLeft( el.offsetParent );
            return offset;
        },
        /**
         * 设置目标样式
         */
        setStyle(el, target) {
            for ( let key in target ) {
                if ( target[key] != null ) el.style[key] = target[key] + 'px';
            }
        },
        /**
         * data 转 rgba 数据
         * @returns { String } 返回 rgba 颜色值 'rgba(0, 0, 0, 1)'
         */
        getColor(data) {
            return `rgba(${data[0]}, ${data[1]}, ${data[2]}, ${Math.round((data[3] / 255) * 100) / 100})`;
        },
        /**
         * 设置拖动圆的 top 和 left 值, 并获取颜色值
         */
        setCircleTopLeft(res) {
            let circle = [res.x + this.circleR, res.y + this.circleR];
            this.setStyle( this.$refs.circle, {
                top  : res.y + this.padding - this.cw / 2,
                left : res.x + this.padding - this.cw / 2,
            } );

            /* update color */
            this.queue.then( _ => {

                // 解决 initColor 与 color 误差的问题
                if ( flag ) {
                    this.commitColor(this.initColor);
                    flag = false;
                    return;
                }

                /* 缓存机制 */
                res.x = Math.round(res.x);
                res.y = Math.round(res.y);
                let data = cache[ res.x + '-' + res.y ];
                if ( data == null ) {
                    cache[ res.x + '-' + res.y ] = ( data = this.getColor( this.ctx.getImageData(...circle, 1, 1).data ) );
                }

                this.commitColor(data);
            } );
        },
        /**
         * 获取样式值
         */
        getStyle(el, target) {
            return Number.parseFloat( window.getComputedStyle( el, null )[target] );
        },

        /* 判断 2 个颜色是否相等 */
        isEqualColor(color1, color2) {
            if ( color1 || color2 ) return false;

            color1 = color1.replace( /\s*/g, '' );
            color2 = color2.replace( /\s*/g, '' );
            return color1 == color2;

            // let flag = true;
            // color1   = color1.match( reg ).splice( 1, 4 );
            // color2 = color2.match( reg ).splice( 1, 4 );
            // color1.forEach( ( el, index ) => {
            //     if ( +el != +color2[index] ) flag = false;
            // } );
            // return flag;
        },

        /* normal Circle */
        norMalCircle() {
            this.setStyle( this.$refs.circle, {
                width  : def.normalW,
                height : def.normalW,
            } );
        },

        /* big Circle */
        bigCircle() {
            this.cw = def.bigW - this.circleW;
            this.setStyle( this.$refs.circle, {
                width  : def.bigW,
                height : def.bigW,
            } );
        }
    },

    watch: {

        'initColor'(val, oldVal) {
            if ( this.isEqualColor( val, this.color ) ) return;
            flag = true;
            this.getInitPoint( val );
        }
    },
};
</script>


<style scoped lang="less">

/* 定位 */
.setPosition( @positon: absolute, @top: 0, @left: 0 ) {
    position: @positon;
    top: @top;
    left: @left;
}

/* 居中 */
.setCenter(){
    margin: 50% 50%;
    transform: translateY( -50% ) translateX( -50% );
}

.colorPicker {
    overflow: hidden;

    /* 内部 */
    .picker {

        .setPosition( relative );
        background-color: transparent;
        margin: auto;
        padding: 10px;

        /* 遮罩层 */
        .mask {

            .setPosition();
            .setCenter();
            border-radius: 50%;
            z-index: 21;
        }

        /* 拖动的圆 */
        .circle {

            box-sizing: border-box;
            .setPosition();
            border: 2px solid #fff;
            border-radius: 50%;
            z-index: 20;
        }

        /* canv */
        .canv {
            opacity: 0;
            .setPosition( relative );
            z-index: 11;
        }

        /* 图片遮罩 */
        .img {
            .setPosition( absolute, 50%, 50% );
            transform: translateY( -50% ) translateX( -50% );
            z-index: 1;
        }
    }
}
</style>