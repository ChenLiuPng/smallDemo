<template>
  <div class="viewport" ref="viewport" @scroll="scrollFn">
    <!-- 自己做一个滚动条 -->
    <div class="scroll-bar" ref="scrollBar"></div>
    <!-- 真实渲染的内容 -->
    <div class="scroll-list" :style="{transform:`translate3d(0,${offset}px,0)`}">
      <div v-for="(item) in visibleData" :vid="item.id" :key="item.id" ref="items">
        <slot :item="item"></slot>
      </div>
    </div>
  </div>
</template>

<script>
import throttle from 'loadsh/throttle'
export default {
  props: {
    size: Number, // 当前每一项的高度
    remain: Number, // 可见多少个
    items: Array,
    variable:Boolean
  },
  data() {
    return {
      start: 0,
      end: this.remain, //默认显示8个
      offset:0
    };
  },
  created(){
      this.scrollFn  = throttle(this.handleScroll,200,{leading:false});
  },
  updated() {
      // 页面渲染完成后 需要根据当前展示的数据 更新缓存区得数据
      this.$nextTick(()=>{
          // 根据当前显示得 更新缓存中得 height bottom top . 最终更新滚动得高度
          let nodes = this.$refs.items; // 获取真实的节点
          if(nodes&&nodes.length>0){
              return;
          }
          nodes.forEach(node=>{
              let {height} = node.getBoundingClientRect(); // 真实的高度
              // 我希望更新老的高度
              let id = node.getAttribute('vid') - 0;
               let oldHeight = this.positions[id].height;
               let val = oldHeight - height; //  计算当前的高度是否和之前的有变化
               if(val) {
                   this.positions[id].height = height;
                   this.positions[id].bottom = this.positions[id].bottom - val; // 底部增加了
                   for(let i = id + 1; i <  this.positions.length;i++) {
                       this.positions[i].top = this.positions[i-1].bottom;
                       this.positions[i].bottom = this.positions[i].bottom -val;
                   }
               }

          });
          this.$refs.scrollBar.style.height = this.positions[this.positions.length-1].bottom + 'px';
      })
  },
  computed: {
      // 渲染三屏
      prevCount() { // 前面预留几个
         return Math.min(this.start,this.remain);
      },
      nextCount() { // 后面预留几个
        return Math.min(this.remain,this.items.length - this.end);
      },
    // 可见的数据有那些
    visibleData() {
        let start = this.start - this.prevCount;
        let end = this.end + this.nextCount;
      return this.items.slice(start, end);
    },
  },
  mounted() {
    this.$refs.viewport.style.height = this.size * this.remain + "px";
    this.$refs.scrollBar.style.height = this.items.length * this.size + "px";

    // 如果加载完毕 我需要缓存每一项的高度
    // 先记录好，等一会滚动得时候 去渲染页面时获取真实dom的高度 来更新缓存的内容
    // 再重新计算滚动条的高度
    this.cacheList();
  },
  methods:{
      getStartIndex(value) { // 查找当前滚动的需要找到的值
          let start = 0; // 开始位置
          let end = this.positions.length - 1; // 结束位置
          let temp = null;
          while(start<=end) {
              let middleIndex = parseInt((start+end)/2);
              let middleValue = this.positions[middleIndex].bottom; // 找到当前的中间的那个人的结尾点
              if(middleValue==value){ // 如果直接找到了 就返回当前的下一个人
                return middleIndex + 1;
              }else if(middleValue < value) { // 当前要查找的人 在右边
                    start = middleIndex + 1
              }else if(middleValue > value) { // 当前要查找的人 在左边
              if(temp==null || temp > middleIndex){
                  temp = middleIndex;
              }
                    end = middleIndex -1;
              }
          }
          return temp;


      },
      handleScroll() {
          // 1. 应该先算出来当前滚过去几个了，当前应该从第几个开始
          let scrollTop = this.$refs.viewport.scrollTop;
          if(this.variable){
              // 如果有传递variable 说明要使用二分查找找到对应的记录
              this.start = this.getStartIndex(scrollTop);
              this.end = this.start + this.remain;
              this.offset = this.positions[this.start - this.prevCount]?this.positions[this.start - this.prevCount].top:0;
          }else{
              // 获取当前应该从第几个开始渲染
                this.start = Math.floor(scrollTop / this.size)
                this.end = this.start + this.remain; // 当前可渲染的区域
                // 定位当前可视化区域
                this.offset = this.start * this.size - this.size * this.prevCount;
          }
          
      },
      cacheList() { // 缓存当前项的高度 和 top值还有bottom
        this.positions = this.items.map((item,index)=>({
            height:this.size,
            top:index * this.size,
            bottom:(index+1) * this.size
        }));
        console.log(this.positions)
      }
  }
};
</script>

<style lang="scss">
.viewport {
  overflow-y: scroll;
  position:relative;
}
.scroll-list {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
}
</style>