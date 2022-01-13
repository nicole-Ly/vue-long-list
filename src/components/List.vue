<template>
  <div class="container"
  ref="container"
   :style="{height: containerHeight}"
   @scroll="handleScroll">
    <div class="scroll-box" :style="{top:top}">
      <div class="item" ref="items" :myid="item.id" v-for="(item,index) in renderItems" :key="index" :style="{minHeight: itemHeight}" :data-id="item.id">
        {{item.content}}
      </div>
    </div>
    <div class="expend" :style="{height: totalHeight}">
    </div>
  </div>
</template>
<script>
/**
 * 当每项高度不固定的情况
 * 1. 定义positions，记录每项头部相对顶部的距离：top  高度：height  尾部相对顶部的距离：bottom  {top:'',height:'',bottom:''}
 * 2. 初始化渲染完成，通过node.getBoundingClientRect在update中拿到每项postion:  {top:'',height:'',bottom:''}
 * 3. 滚动时，根据二分查找法对比距离顶部top和中间项的bottom，来根据获取start
 * 4. 根据start和end截取展示的数据范围
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
    },
    //是否为可变高度
    variable:{
      type:Boolean,
      value:false
    }
  },
  data(){
    return {
      start:0,
      end:this.showNumber,
      positions:[]
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
      if(this.variable){
        //最后一个元素的bottom
        let lastItem = this.positions[this.positions.length-1];
        return `${lastItem?lastItem.bottom:0}px`;
      }else{
        return `${this.items.length * this.size}px`;
      }
    },
    //处理超出顶部边界滚动断层情况
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
      //当设置了预加载的结构时，top需要响应的减去这部分尺寸
      if(this.variable){
        return `${this.positions[this.start - this.prev] ? this.positions[this.start - this.prev].top:0}px`
      }else{
        return`${(this.start - this.prev) * this.size}px`;
      }
    }
  },
  created(){
    this.cachePostion();
  },
  mounted(){
    this.$nextTick(() => {
      console.log('render')
      let nodes = this.$refs.items
      if(!nodes||!nodes.length) return 
      nodes.forEach(node=>{
        const {height} = node.getBoundingClientRect();
        const id = node.getAttribute('myid') - 0
        console.log("33333",this.positions,id)

        const oldHeight = this.positions[id].height||0;
        const val = height - oldHeight;
        if(val){//如果有变化，更新当前元素以及后续所有元素的坐标
          this.positions[id].height = height;//当前元素高度变化只对自身bottom、后续元素的top和bottom有影响
          this.positions[id].bottom = this.positions[id].bottom + val;
          for (let i = id + 1; i < this.positions.length; i++) {//修改当前元素以后的元素
            this.positions[i].top = this.positions[i - 1].bottom
            this.positions[i].bottom = this.positions[i].bottom + val
          }
        }
      })
    })
  },
  methods:{
    cachePostion(){
      const size = this.size;
      this.positions = this.items.map((item,index)=>({
        top:index * size,
        height:size,
        bottom:(index+1)*size
      }))
    },
    /* 页面滚动时
     * 根据距离顶部高度，计算开始和结束索引
     * 从而截取对应区间的数据列表
     */
    handleScroll(){
      const scrollTop = this.$refs.container.scrollTop
      if(this.variable){
        this.start = this.getStartIndex(scrollTop);
        this.end = this.start + this.showNumber; 
      }else{
        this.start = Math.floor(scrollTop/this.size);
        this.end = this.start + this.showNumber;  
      } 
    },
    /**
     *二分查找法
     * 匹配postions中bottom和scrollTop相等项，从而查找获取对应start
     */
    getStartIndex(scrollTop){
      let start = 0;
      let end = this.positions.length-1;
      let index = 0;
      while(start<=end){
        let middleIndex = parseInt((start+end)/2);
        let middleValue = this.positions[middleIndex].bottom;
        if(middleValue == scrollTop){
          return middleIndex + 1;
        }else if(middleValue > scrollTop){
          end = middleIndex - 1;
        }else if(middleValue<scrollTop){
          start = middleIndex +1;
        }
        index = middleIndex;
      }
      return index;
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