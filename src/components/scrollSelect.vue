<template>
  <section class="scroll-select">
    <div class="scroll-select-data">
      滚轮选择器选中的是：{{addressDate.province}}-{{addressDate.city}}-{{addressDate.area}}
    </div>
    <div class="scroll-select-box">
      <div class="scroll-select-box-btn">
        <div @click="$emit('click')">取消</div>
        <div @click="determine">确定</div>
      </div>
      <div class="scroll-select-list">
        <scroll-select-list :listData="listData"></scroll-select-list>
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

    &-btn {
      border-top: 1px solid #dcdcdc;
      border-bottom: 1px solid #dcdcdc;
      display: flex;
      padding: 0 12px;
      justify-content: space-between;
      align-items: center;
      height: 50px;
      font-size: 18px;
      color: #3f91f6;
      box-sizing: border-box;
    }
  }
}
</style>
