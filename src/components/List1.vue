<template>
  <div class="container"
  ref="container"
   :style="{height: containerHeight}"
   @scroll="handleScroll">
    <div class="scroll-box" :style="{top:top}">
      <div class="item" v-for="(item,index) in renderItems" :key="index" :style="{height: itemHeight}" :data-id="item.id">
        {{item.content}}
      </div>
    </div>
    <div class="expend" :style="{height: totalHeight}">
    </div>
  </div>
</template>
<script>
/**
 * 当每项高度固定的情况
 * 1. 滚动时获取距离顶部top，通过top和size计算出start和end
 * 2. 根据start和end截取需显示的数据范围
 * 3. 优化首尾边界断层
 */
export default{
  props:{
     //全部数据
    items:{
      type:Array,
      default:()=>{
        return []
      }
    },
    //视口显示条数
    showNumber:{
      type:Number,
      default:10
    },
    //每条高度
    size:{
      type:Number,
      default:50
    }
  },
  data(){
    return {
      start:0,
      end:this.showNumber
    }
  },
  computed:{
    //每项高度
    itemHeight(){
      return `${this.size}px`
    },
    //显示区域高度
    containerHeight(){
      return `${this.showNumber * this.size}px`;
    },
    //总高度
    totalHeight(){
      return `${this.items.length * this.size}px`;
    },
    //处理临界断层情况
    prev(){
      return Math.min(this.start,10)
    },
    //页面渲染项
    renderItems(){
      // 为了能够在空闲时多加载一些数据，例如前面10个与后面10个，目的是减少页面与数据的更新
      // 为了避免超出边界，需要进行边界值检测
      // start 最小值为0
      const start = this.start - this.prev;
      const end = this.end + Math.min(10, this.items.length - this.end);
      return this.items.slice(start,end)
    },
    // 滚动中，实时修改 list 的 top ，用于填补滚动条的空白
    top(){
      //当设置了预加载的结构时，top 需要响应的减去这部分尺寸
      return`${(this.start - this.prev) * this.size}px`;
    }
  },
  mounted(){

  },
  methods:{
    /* 页面滚动时
     * 根据距离顶部高度，计算开始和结束索引
     * 从而截取对应区间的数据列表
     */
    handleScroll(){
      const scrollTop = this.$refs.container.scrollTop
      this.start = Math.floor(scrollTop/this.size);
      this.end = this.start + this.showNumber;  
    }
  }
}
</script>
<style scoped>
  .container{
    position: relative;
    border:3px solid black;
    padding: 0 10px;
    overflow-y: scroll;
  }
  .scroll-box{
    width: 100%;
    position: absolute;
  }
  .item{
    text-align: left;
    border-bottom:1px solid gray;
  }
</style>