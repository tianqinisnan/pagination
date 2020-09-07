<template>
  <div class="home">
    <div class="set-total">
      <span @click="setTotal(0)">total: 0</span>
      <span @click="setTotal(4)">total: 4</span>
      <span @click="setTotal(10)">total: 10</span>
      <span @click="setTotal(12)">total: 12</span>
      <span @click="setTotal(30)">total: 30</span>
      <span @click="setTotal(50)">total: 50</span>
      <span @click="setTotal(58)">total: 58</span>
      <span @click="setTotal(102)">total: 102</span>
    </div>
    <Pagination 
      :total="totalCount" 
      :pageSize="5" 
      :pagerCount="10" 
      :ellipsisCanJump="true" 
      :needJump="true"
      :needPrevNext="true"
      @currentChange="currentChange">
    </Pagination>
  </div>
</template>

<script>
import Pagination from '@/components/Pagination.vue'
export default {
  name: 'Home',
  components: {
    Pagination
  },
  data() {
    return {
      totalCount: 0
    }
  },
  created() {
    setTimeout(_ => {
      this.totalCount = 102
    }, 2000)
  },
  methods: {
    currentChange(currentPage) {
      console.log(currentPage)
    },
    setTotal(num) {
      this.totalCount = num
    }
  }
}
</script>

<style lang="less" scoped>
.set-total {
  margin-bottom: 10px;
  span {
    cursor: pointer;
    margin: 0 10px;
    &:hover {
      color: #db2c17;
    }
  }
}
</style>
