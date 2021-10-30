<template>
  <div id="app">
    <!-- 我需要只显示可视区域 -->
    <!-- 1.告诉我的列表每一项多高 因为我要算出一个滚动条来 -->
    <!-- 2.variable 这个高度不一定多高了 -->
    <virtual-list :size="100" :remain="8" :items="items" :variable="true">
      <Item slot-scope="{item}" :item="item" />
    </virtual-list>
  </div>
</template>

<script>
import VirtualList from './components/virtual-list.vue'
import Item from './components/item';
import Mock from 'mockjs';
let items = [];
for(let i = 0; i< 10000; i++) {
  items.push({id:i,value:Mock.Random.sentence(5,20)});
}
export default {
  name: 'App',
  components: {
    VirtualList,
    Item
  },
  data() {
    return {
      items
    }
  }
}
</script>

<style lang="scss">
* {
  margin: 0;
  padding:0;
  box-sizing: border-box;
}
</style>
