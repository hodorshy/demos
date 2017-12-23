<template>
  <div class="lightstair" ref="lightStair">
    <div class="inner">
      <ul ref="ul">
        <li v-for="(item,index) in lightPic" :key="index" :class="{active: isSmall}">
          <img :class="'lightItem' + '_' + index" :src="item.pic"
              v-show="getCount() > (lightPic.length - 1 - index) ? true : false" alt="图片路径错误">
          <img :class="'lightItem' + '_' + index" :src="item.picOff"
              v-show="getCount() > (lightPic.length - 1 - index) ? false : true" alt="图片路径错误">
        </li>
      </ul>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">

  /**
   * <LightStair id="lightStair" :lightPic="lightPic" :callback="callback" :event-target="target" :lightCount.sync="lightCount"></LightStair>
   * <LightStair id="lightStair" :lightPic="lightPic" :callback="callback" :lightCount.sync="lightCount"></LightStair>
   *
   *  // 图片路径                             (必须)
   *  lightPic: [{
   *      "pic": "../../static/images/on/image_lamp_light_010.png",                       // on 图片路径
   *      "picOff": "../../static/images/off/image_lamp_light_010_press.png",             // off 关闭
   *  }]
   *
   * lightCount     返回亮灯的个数             (必须, 会跟新返回)
   *
   * callback(this) 回调函数                  (可选)
   *
   * eventTarget    滑动时间源, 没有组织冒泡   (可选, 默认是这个组件的范围)
   *
   * isAction       是否在滑动                (可选)
   *
   */

  let def = {
    start  : 0,
    end    : 100,
    offset : 3,
    step   : 3.5,
    count  : 0,
    width  : undefined,
    height : undefined,
    scaleW : .22,
    scaleH : 3.1,
    scale  : .5,
  };

  let startx, starty, endx, endy, direction, oldValue, step;

  export default {
    props: ["lightPic", "callback", "lightCount", "isRender", "isSmall"],
    data() {
      return {
        curVal: 0,
        isAction: false,
      };
    },
    mounted() {
      let target = this.$refs.lightStair.parentNode;

      target.addEventListener('touchstart', this.touchStart);
      target.addEventListener('touchmove', this.touchMove);
      target.addEventListener('touchend', this.touchEnd);

      this.curVal = this.initCount(this.lightCount);
    },
    created() {
      def.count = this.lightPic.length;
      step = (def.end - def.start) / def.count;
    },
    watch: {
      isAction(val) {
        // this.$emit('upadte:isAction', this.isAction);
      },
      lightCount(val) {
        this.curVal = this.initCount(val);
      },
    },
    methods: {
      touchStart(e) {

        startx = e.touches[0].pageX;
        starty = e.touches[0].pageY;
        oldValue = this.curVal;
      },
      touchMove(e) {

        if (this.isRender) return;

        this.isAction = true;
        endx = e.changedTouches[0].pageX;
        endy = e.changedTouches[0].pageY;
        direction = this.getDirection(startx, starty, endx, endy);
        /* 判断方向 */
        switch (direction) {
          case 0:     // 未滑动！
            break;
          case 1:     // 向上！
            this.curVal += def.step;
            break;
          case 2:     // 向下！
            this.curVal -= def.step;
            break;
          case 3:     // 向左！
            break;
          case 4:     // 向右！
            break;
          default:
        }
        if (this.curVal >= def.end + def.offset) this.curVal = def.end;
        if (this.curVal <= def.start - def.offset) this.curVal = def.start;
        startx = endx;
        starty = endy;
      },
      touchEnd(e) {

        // if (Math.abs(this.curVal - oldValue) >= 1) typeof this.callback === 'function' && this.callback(this);

        // 更新
        if (Math.abs(this.curVal - oldValue) >= 1) this.$emit('update:lightCount', this.getCount());
        this.isAction = false;
      },
      getAngle(angx, angy) {
        return Math.atan2(angy, angx) * 180 / Math.PI;
      },
      getDirection(startx, starty, endx, endy) {
        var angx = endx - startx;
        var angy = endy - starty;
        var result = 0;
        //如果滑动距离太短
        if (Math.abs(angx) < 2 && Math.abs(angy) < 2) {
          return result;
        }
        var angle = this.getAngle(angx, angy);
        if (angle >= -135 && angle <= -45) {
          result = 1;
        } else if (angle > 45 && angle < 135) {
          result = 2;
        } else if ((angle >= 135 && angle <= 180) || (angle >= -180 && angle < -135)) {
          result = 3;
        } else if (angle >= -45 && angle <= 45) {
          result = 4;
        }
        return result;
      },
      getCount() {

        return Math.round(this.curVal / step);
      },
      initCount(val) {
        return val * step;
      },
    },
  };
</script>

<style scoped>
  .lightstair {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  .lightstair ul.active {
    width: 60px;
    height: 182px;
  }

  .lightstair li.active {
    width: 60px;
    height: 15px;
    margin-top: 3px;
  }

  .lightstair ul {
    width: 80px;
    height: 202px;
  }

  .lightstair li {
    width: 80px;
    height: 19px;
    margin: auto;
    margin-top: 4px;
  }

  .lightstair li:last-of-type {
    height: 24px;
  }
  .lightstair li:first-of-type {
    height: 26px;
  }

  .lightstair li.active:last-of-type {
    height: 16px;
  }
  .lightstair li.active:first-of-type {
    height: 19px;
  }

  .lightstair li img {
    width: 100%;
    height: 100%;
    position: relative;
  }

</style>