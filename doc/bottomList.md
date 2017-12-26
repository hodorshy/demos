```js


根据 word 的变化来响应数据, img 的变化都要根据 word 来变化 +
考虑到动画, 所以 active 不需要绑定, 自定修改就好 +
curIndex 与 active 的绑定的思考, 监听 active +
  提供一个 active 的触发事件 +
curIndex 与 isSwitch 和 childMode 不绑定
模拟上报打印
不传入 curIndex  就是取消默认控制 +
传入 curIndex 和 index 受默认行为控制 +
props-slot
1. 自定义童锁
2. zjf 案例, 需要图片
3. simple
4. 灯关案例


[
  	// 根据 data[] 来计算
  { group: false, switch: this.switch, word: this.word, isChild: false }
]
data: []
imgUrl: []
img_on: '', img_off: ''	// 默认值为当前路径下的图片
computed: {
    words() { return [] } // 使用 slot | filter 来显示
}

// 每个都写成一个子组件
// 可以使用 router-link 来使用


.active {
 	opacity: 1;	// 改变透明度
}

slot // 特殊使用 以及 word 来绑定

curIndex.sync
-$index // 为 active
Math.abs( index ) === $index ? -index : $index; // if ( !data[ $index ].group )
index = $index;
// null 空 其他代表位置为 active
// select 单独使用

opt: {
  data: [ { group: false, custom: false, word: undefined /* 在created创建 */,
          callback: fun, } ],
  imgUrl: [ ],
  words: [ ], // 计算
  childMode: undefined,
  isSwitch: 0,
  img_on: // 默认值
  img_off: // 默认值
  curIndex.sync: // 默认值
}

// callbackCild 可以自定义 childMode 自定义行为
// slot 自定义
// flag + curIndex 决定 active 类
// 单页禁止滑动, 不显示分页点
// group === false 时候, 默认保留上次状态, 添加 oldValue 属性
// 开窗的那个案例可以使用 curIndex 来监听
```


