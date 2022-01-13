<template>
  <div id="app">
    <List
      :items="items"
      :size="70"
      :showNumber="10"
      :variable="true"
    ></List>
  </div>
</template>

<script>
import Mock from 'mockjs'
import List from './components/List.vue'
/**
 * 列表组件需要哪些参数
 * 长列表数据
 * 每项高度
 * 当前页显示条数
 */
export default {
  name: 'App',
  components: { 
    List
  },
  computed: {
    items () {
      return Array(10000).fill('').map((item, index) => ({
        id: index,
        // content: `列表项的内容：${index}`
        content: Mock.Random.sentence(1, 10)
      }))
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
