<template>
  <div class="app">
    <v-header :seller="seller"></v-header>
    <div class="tab border-1px">
      <div class="tab-item">
        <a v-link="{path:'/goods'}">商品</a>
        </div>
      <div class="tab-item">
        <a v-link="{path:'/ratings'}">评论</a>
        </div>
      <div class="tab-item">
        <a v-link="{path:'/seller'}">商家</a>
        </div>
    </div>
    <router-view :seller="seller" keep-alive></router-view>
  </div>
</template>
<script type="text/ecmascript-6">
  import header from 'components/header/header.vue'
  import { urlParse } from 'common/js/util'

  const ERR_OK = 0

  export default {
    data() {
      return {
        seller: {
          id: (() => {
            let queryParam = urlParse()
            return queryParam.id
          })()
        }
      }
    },
    created() {
      this.$http.get('/api/seller?id=' + this.seller.id).then(res => {
        res = res.body
        if (res.errno === ERR_OK) {
          this.seller = Object.assign({}, this.seller, res.data)
          // assign是es6的语法，可以扩展对象属性
          console.log(this.seller)
        }
      })
    },
    components: {
      'v-header': header
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "./common/stylus/mixin.styl"
  .app
    .tab
      display: flex
      width: 100%
      height: 40px
      line-height: 40px
      // border-bottom: 1px solid rgba(7, 17, 27, 0.1)
      border-1px(rgba(7, 17, 27, 0.1))
      .tab-item
        flex: 1
        text-align: center
        & > a
          display: block
          font-size: 16px
          color: rgb(77, 85, 93)
          &.active
            color: rgb(240, 20, 20)
</style>
