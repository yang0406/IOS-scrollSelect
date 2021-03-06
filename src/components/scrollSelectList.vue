<template>
  <div class="scroll-select-item">
    <div class="scroll-select-line"></div>
    <div class="scroll-select-list">
      <ul class="scroll-select-ul" ref="list">
        <li class="scroll-select-list-item" v-for="(el,index) in renderData " :class="{'hidden':setHidden(el.index)}"
            :key="index">{{el.value}}
        </li>
      </ul>
    </div>
    <ul class="scroll-select-wheel" ref="wheel">
      <li class="scroll-select-wheel-item" :class="{'hidden':setHidden(el.index)}" :style="setWheelItemDeg(el.index)"
          :index="el.index" v-for="(el,index) in renderData " :key="index">{{el.value}}
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    name: 'scrollSelectList',

    data() {
      return {
        spin: {start: -9, end: 9, branch: 9},
        finger: {startY: 0, lastY: 0, startTime: 0, lastTime: 0, transformY: 0},
        baseSize: '',
      }
    },

    props: {
      listData: {
        type: Array,
        required: true,
      },
      type: {
        type: String,
        default: 'line',
      },
    },

    computed: {
      renderData() {
        let temp = [];
        for (let k = this.spin.start; k <= this.spin.end; k++) {
          const data = {
            value: this.getSpinData(k),
            index: k,
          }
          temp.push(data);
        }
        return temp;
      }
    },

    mounted() {
      // 事件绑定
      this.$el.addEventListener('touchstart', this.itemTouchStart);
      this.$el.addEventListener('touchmove', this.itemTouchMove);
      this.$el.addEventListener('touchend', this.itemTouchEnd);
      this.init();
    },

    methods: {
      // 初始化
      init() {
        this.baseSize = 16 * Math.min((document.documentElement.clientWidth / 375), 2);
        this.$nextTick(()=>{
          this.setListTransform();
          this.getPickValue(0);
        })
      },

      // 根据type 控制滚轮显示效果
      setHidden(index) {
        if (this.type === 'line') {
          return index < 0 || index > this.listData.length - 1;
        } else {
          return false;
        }
      },

      // 初始化选中样式
      setWheelItemDeg(index) {
        return {
          transform: `rotate3d(1, 0, 0, ${-index * 20 % 360}deg) translate3d(0px, 0px, 6.25rem)`
        }
      },

      setWheelDeg(updateDeg, type, time = 1000) {
        if (type === 'end') {
          this.$refs.wheel.style.webkitTransition = `transform ${time}ms cubic-bezier(0.19, 1, 0.22, 1)`;
          this.$refs.wheel.style.webkitTransform = `rotate3d(1, 0, 0, ${updateDeg}deg)`;
        } else {
          this.$refs.wheel.style.webkitTransition = '';
          this.$refs.wheel.style.webkitTransform = `rotate3d(1, 0, 0, ${updateDeg}deg)`;
        }
      },

      // 滚动时样式变化
      setListTransform(translateY = 0, marginTop = 0, type, time = 1000) {
        if (type === 'end') {
          this.$refs.list.style.webkitTransition = `transform ${time}ms cubic-bezier(0.19, 1, 0.22, 1)`;
          this.$refs.list.style.webkitTransform = `translateY(${translateY / this.baseSize - this.spin.branch * 2.125}rem)`;
          this.$refs.list.style.marginTop = `${-marginTop / this.baseSize}rem`;
          this.$refs.list.setAttribute('scroll', translateY);
        } else {
          this.$refs.list.style.webkitTransition = '';
          this.$refs.list.style.webkitTransform = `translateY(${translateY /this.baseSize - this.spin.branch * 2.125}rem)`;
          this.$refs.list.style.marginTop = `${-marginTop / this.baseSize}rem`;
          this.$refs.list.setAttribute('scroll', translateY);
        }
      },

      // 触摸开始事件
      itemTouchStart(event) {
        this.finger.startY = event.changedTouches[0].pageY;
        this.finger.startTime = event.timeStamp;
        this.finger.transformY = this.$refs.list.getAttribute('scroll');
        // 取消默认事件
        event.preventDefault();
      },

      // 触摸滑动事件
      itemTouchMove(event) {
        this.finger.lastY = event.changedTouches[0].pageY;
        this.finger.lastTime = event.timeStamp;
        // 设置css
        const move = this.finger.lastY - this.finger.startY;
        this.setStyle(move);
        event.preventDefault();
      },

      // 触摸结束事件
      itemTouchEnd(event) {
        this.finger.lastY = event.changedTouches[0].pageY;
        this.finger.lastTime = event.timeStamp;
        let move = this.finger.lastY - this.finger.startY;
        // 计算速度
        // 速度计算说明
        // 当时间小于300毫秒 最后的移动距离等于 move + 减速运动距离
        let time = this.finger.lastTime - this.finger.startTime;
        let v = move / time;
        // 减速加速度a(自己设置)
        let a = 3;
        // 设置css
        if (time <= 300) {
          move = v * a * time;
          time = 1000 + time * a;
          this.setStyle(move, 'end', time);
        } else {
          this.setStyle(move, 'end');
        }
      },

      // 设置css
      setStyle(move, type, time) {
        // 单个li高度
        const singleHeight = 2.125 * this.baseSize;
        const deg = 20;
        const singleDeg = deg / singleHeight;
        let currentListMove = this.finger.transformY;
        let updateMove = move + Number(currentListMove);
        let updateDeg, spinAim, margin, endMove, endDeg;
        if (type === 'end' && this.type === 'line') {
          // 这里只在释放的时候判断 实现缓动效果
          // 根据滚轮类型 line or cycle 判断 updateMove最大距离
          if (updateMove > 0) {
            updateMove = 0;
          }
          if (updateMove < -(this.listData.length - 1) * singleHeight) {
            updateMove = -(this.listData.length - 1) * singleHeight;
          }
        }
        //todo 这里考虑后续设置能最大缓动的值 目前暂时不考虑

        updateDeg = -updateMove * singleDeg;
        spinAim = Math.round(updateDeg / 20);
        margin = Math.round(updateMove / singleHeight) * singleHeight; // 如果不这么写 会导致没有滚动效果
        /* 计算touchEnd移动的整数距离 */
        endMove = margin;
        endDeg = Math.round(updateDeg / deg) * deg;
        if (type === 'end') {
          this.setListTransform(endMove, margin, type, time);
          this.setWheelDeg(endDeg, type, time);
          /* 设置$emit 延迟 */
          setTimeout(() => this.getPickValue(endMove), 1000);
        } else {
          this.setListTransform(updateMove, margin);
          this.setWheelDeg(updateDeg);
        }
        this.updateSpin(spinAim);
      },

      // 更新spin
      updateSpin(spinAim) {
        this.spin.start = this.spin.branch * -1 + spinAim;
        this.spin.end = this.spin.start + this.spin.branch * 2;
      },

      // 获取spin 数据
      getSpinData(index) {
        index = index % this.listData.length;
        return this.listData[index >= 0 ? index : index + this.listData.length];
      },

      // 获取选中值
      getPickValue(move) {
        const index = Math.round(-move / (2.125 * this.baseSize));
        const pickValue = this.getSpinData(index);
        this.$emit('pickSelect', pickValue);
      }
    },
    beforeDestroy() {
      this.$el.removeEventListener('touchstart', this.itemTouchStart);
      this.$el.removeEventListener('touchmove', this.itemTouchMove);
      this.$el.removeEventListener('touchend', this.itemTouchEnd);
    }
  }
</script>

<style lang="less" scoped>
.scroll {
  &-select {
    &-item {
      overflow: hidden;
      width: 100%;
      text-align: center;
      height: 220px;
      background: #fff;
      position: relative;
      & ul, li {
        padding: 0;
        list-style: none;
        margin: 0;
      }
    }
    &-ul {
      position: relative;
    }
    &-line, &-list, &-wheel {
      position: absolute;
      left: 0;
      right: 0;
      top: 93px;
    }
    &-line {
      z-index: 3;
    }
    &-list {
      z-index: 2;
      background: #fff;
    }
    &-wheel {
      z-index: 1;
    }
    &-line {
      &:after, &:before {
        position: absolute;
        top: 0;
        content: '';
        display: table;
        background: #2c97f1;
        width: 100%;
        height: 2px;
        -webkit-transform: scaleY(0.5);
        transform: scaleY(0.5);
        -webkit-transform-origin: 0 0;
        transform-origin: 0 0;
      }
      &:before {
        bottom: -1px;
        top: auto;
      }
    }
    &-line, &-list {
      height: 34px;
      transform: translate3d(0px, 0px, 110px);
    }
    &-list {
      overflow: hidden;
    }
    &-list-item {
      text-shadow: 0 1px 1px rgba(102, 102, 102, 0.6);
    }
    &-list-item, &-wheel-item {
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      line-height: 34px;
      font-size: 18px;
      color: #333;
      &.hidden {
        visibility: hidden;
        opacity: 0;
      }
    }
    &-wheel {
      transform-style: preserve-3d;
      height: 34px;
      &-item {
        backface-visibility: hidden;
        position: absolute;
        top: 0;
        width: 100%;
        color: #a8a8a8;
      }
    }
  }
}
</style>
