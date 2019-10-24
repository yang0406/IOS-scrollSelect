<template>
  <section class="scroll-select">
    <div class="scroll-select-data">
      滚轮选择器选中的是：{{addressDate.province}}-{{addressDate.city}}-{{addressDate.area}}
    </div>
    <div class="scroll-select-box">
      <div class="scroll-select-box-title">滚筒选择器</div>
      <div class="scroll-select-list">
        <scroll-select-list :listData="listData" @pickSelect="pickSelect($event, 'first')"></scroll-select-list>
        <scroll-select-list :listData="listData" @pickSelect="pickSelect($event, 'second')"></scroll-select-list>
        <scroll-select-list :listData="listData" @pickSelect="pickSelect($event, 'third')"></scroll-select-list>
      </div>
    </div>
  </section>
</template>

<script>
import scrollSelectList from './scrollSelectList';
export default {
  name: 'scrollSelect',

  data() {
    return {
      listData: Array.from({length: 12}, (value, index) => 1 + index),
      addressDate: {
        province: '1',
        city: '2',
        area: '3',
      }
    };
  },

  components: {
    scrollSelectList,
  },

  methods: {
    determine() {
      console.log(this.addressDate);
    },

    pickSelect(val, type) {
      if (type === 'first') {
        this.addressDate.province = val;
      } else if (type === 'second') {
        this.addressDate.city = val;
      } else if (type === 'third') {
        this.addressDate.area = val;
      }
    },
  },
}
</script>

<style lang="less">
.scroll-select {
  &-box {
    position: fixed;
    bottom: 0;
    left: 0;
    width: 100%;
    height: 270px;

    &-title {
      border-top: 1px solid #dcdcdc;
      border-bottom: 1px solid #dcdcdc;
      text-align: center;
      height: 50px;
      line-height: 50px;
      font-size: 18px;
      color: #3f91f6;
      box-sizing: border-box;
    }
  }

  &-list {
    display: flex;
    justify-content: space-around;
  }
}
</style>
