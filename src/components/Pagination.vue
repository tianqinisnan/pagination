<!-- 分页 Pagination -->
<template>
  <div class="pagination-wrapper" v-if="total > 0">
    <button v-if="needPrevNext" class="page-prev" :class="{disabled: prevDisabled}" @click="movePage('prev')">
      <i class="iconfont">&#xe75b;</i>
    </button>
    <ul class="page-list-wrapper">
      <li v-for="page in pages" :key="page.id" :class="{active: page.id === currentPage, disabled: page.ellipsis && !ellipsisCanJump}" @click="selectPage(page)">
        {{page.text}}
      </li>
    </ul>
    <button v-if="needPrevNext" class="page-next" :class="{disabled: nextDisabled}" @click="movePage('next')">
      <i class="iconfont">&#xe75c;</i>
    </button>
    <div class="page-jump-wrapper" v-if="needJump">
      <span>前往</span>
      <input type="number" class="jump-text" autocomplete="off" v-model="jumpPageNum" min="1" :max="pageCount">
      <span>页</span>
      <button class="jump-btn" @click="jumpPage">GO</button>
    </div>
  </div>
</template>

<script>
const BEFORE = Symbol('before')
const AFTER = Symbol('after')
export default {
  props: {
    total: {
      type: Number,
      default: 0
    },
    pageSize: {
      type: Number,
      default: 10
    },
    pagerCount: {
      type: Number,
      default: 7
    },
    needPrevNext: {
      type: Boolean,
      default: false
    },
    needJump: {
      type: Boolean,
      default: false
    },
    ellipsisCanJump: {
      type: Boolean,
      default: true
    }
  },
  components: {
  },
  data () {
    return {
      pageCount: 0, //总页数
      pages: [],  //页码列表
      currentPage: 1, //当前页数
      jumpPageNum: undefined
    }
  },
  created() {
    this.initPages()
  },
  methods: {
    initPages() {
      if (!this.total) return
      this.currentPage = 1
      this.pageCount = ~~(this.total / this.pageSize) + (this.total % this.pageSize ? 1 : 0)
      this.getPageList()
    },
    getPageList() {
      this.pages = []
      let p = 1, pagers = this.needEllipsis ? this.realPagerCount : this.pageCount
      for (let i=1; i<=pagers; i++) {
        let id = p, text = String(p), ellipsis = false
        if (i === 2 && this.beforeEllipsisNum > 1) {
          //第二个设置 '...'
          id = BEFORE
          text = '...'
          ellipsis = true
          p = p + this.countBeforeJump
        } else if (i === pagers - 1 && this.afterEllipsisNum > 1) {
          //倒数第二个设置 '...'
          id = AFTER
          text = '...'
          ellipsis = true
          p = p + this.countAfterJump
        } else {
          p++
        }
        // console.log('i='+i, 'id='+id, 'p='+p)
        this.pages.push({id, text, ellipsis})
      }
    },
    selectPage(page) {
      if (page.id === BEFORE) {
        if (!this.ellipsisCanJump) return
        this.currentPage = 1 + this.countBeforeJump
      } else if (page.id === AFTER) {
        if (!this.ellipsisCanJump) return
        this.currentPage = this.pageCount - this.countAfterJump
      } else {
        this.currentPage = page.id
      }
      this.emitCurrentChange()
    },
    movePage(direction) {
      if (direction === 'prev' && !this.prevDisabled) {
        this.currentPage--
        this.emitCurrentChange()
      }
      if (direction === 'next' && !this.nextDisabled) {
        this.currentPage++
        this.emitCurrentChange()
      }
    },
    jumpPage() {
      if (this.jumpPageNum > this.pageCount) {
        this.jumpPageNum = this.pageCount
      }
      if (this.jumpPageNum < 1) {
        this.jumpPageNum = 1
      }
      this.jumpPageNum = ~~this.jumpPageNum || 1
      this.currentPage = this.jumpPageNum
      this.emitCurrentChange()
    },
    emitCurrentChange() {
      this.getPageList()
      this.$emit('currentChange', this.currentPage)
    }
  },
  watch: {
    total() {
      this.initPages()
    }
  },
  computed: {
    realPagerCount() {
      if (this.pagerCount < 7) {
        console.log('pagerCount 不在7～15区间')
        return 7
      }
      if (this.pagerCount > 15) {
        console.log('pagerCount 不在7～15区间')
        return 15
      }
      return this.pagerCount
    },
    getSidesPageNum() {  //pagerCount均分
      return {
        before: Math.floor((this.realPagerCount-1)/2),
        after: Math.floor(this.realPagerCount/2)
      }
    },
    needEllipsis() {  //是否需要省略页码
      return this.pageCount > this.realPagerCount
    },
    beforeEllipsisNum() { //前面省略的页码数
      if (!this.needEllipsis) return 0
      return this.currentPage - this.getSidesPageNum.before
    },
    afterEllipsisNum() { //后面省略的页码数
      if (!this.needEllipsis) return 0
      return this.pageCount - this.currentPage - this.getSidesPageNum.after + 1
    },
    countBeforeJump() {
      return this.beforeEllipsisNum + (this.afterEllipsisNum < 1 ? this.afterEllipsisNum - 1 : 0)
    },
    countAfterJump() {
      return this.afterEllipsisNum + (this.beforeEllipsisNum < 1 ? this.beforeEllipsisNum - 1 : 0)
    },
    prevDisabled() {
      return this.pageCount <= 1 || (this.currentPage === 1)
    },
    nextDisabled() {
      return this.pageCount <= 1 || (this.currentPage === this.pageCount)
    }
  }
}
</script>

<style lang="less" scoped>
.pagination-wrapper {
  white-space: nowrap;
  padding: 2px 5px;
  color: #303133;
  .page-prev, .page-next {
    margin: 0 5px;
    background-color: #f4f4f5;
    color: #606266;
    min-width: 32px;
    border-radius: 2px;
    border: none;
    display: inline-block;
    font-size: 13px;
    height: 32px;
    line-height: 32px;
    vertical-align: top;
    box-sizing: border-box;
    background-color: transparent;
    border: 1px solid #ddd;
    cursor: pointer;
    &:focus {
      outline: none;
    }
    &.disabled {
      opacity: .5;
      cursor: not-allowed;
    }
  }
  .page-list-wrapper {
    user-select: none;
    list-style: none;
    display: inline-block;
    vertical-align: top;
    font-size: 0;
    padding: 0;
    margin: 0;
    li {
      color: #666;
      padding: 0 4px;
      background-color: transparent;
      vertical-align: top;
      display: inline-block;
      font-size: 13px;
      min-width: 32px;
      border-radius: 2px;
      height: 32px;
      line-height: 32px;
      cursor: pointer;
      box-sizing: border-box;
      text-align: center;
      margin: 0 5px;
      border: 1px solid #ddd;
      &.active {
        color: #db2c17;
        background-color: #fff8f7;
        border-color: #db2c17;
      }
      &.disabled {
        border: none;
        cursor: default;
      }
      &:hover {
        border-color: #db2c17;
      }
    }
  }
  .page-jump-wrapper {
    display: inline-block;
    height: 32px;
    line-height: 32px;
    color: #666;
    margin-left: 20px;
    .jump-text {
      border-radius: 4px;
      border: 1px solid #ddd;
      box-sizing: border-box;
      color: #666;
      display: inline-block;
      font-size: inherit;
      width: 44px;
      height: 32px;
      line-height: 32px;
      outline: none;
      margin: 0 3px;
      padding: 0 6px;
      text-align: center;
      -moz-appearance: textfield;
      &::-webkit-outer-spin-button,
      &::-webkit-inner-spin-button {
        -webkit-appearance: none;
      }
    }
    .jump-btn {
      margin: 0 5px;
      background-color: #f4f4f5;
      color: #606266;
      min-width: 32px;
      border-radius: 2px;
      border: none;
      display: inline-block;
      font-size: 13px;
      height: 32px;
      line-height: 32px;
      vertical-align: top;
      box-sizing: border-box;
      background-color: transparent;
      border: 1px solid #ddd;
      margin-left: 10px;
      cursor: pointer;
      &:focus {
        outline: none;
      }
      &:hover {
        border-color: #db2c17;
      }
    }
  }
}
</style>