<template>
  <div class="food" v-show="showFlag" transition="move" v-el:food>
    <div class="food-content">
      <div class="image-header">
        <img :src="food.image" alt="">
        <div class="back" @click="showFlag = false">
          <i class="icon-arrow_lift"></i>
        </div>
      </div>
      <div class="content">
        <h1 class="title">{{ food.name }}</h1>
        <div class="detail">
          <span class="sell-count">月售{{ food.sellCount }}份</span>
          <span class="rating">好评率{{ food.rating }}%</span>
        </div>
        <div class="price">
          <span class="new">￥{{ food.price }}</span>
          <span class="old" v-show="food.oldPrice">￥{{ food.oldPrice }}</span>
        </div>
        <div class="cartcontrol-warpper">
          <cartcontrol :food="food"></cartcontrol>
        </div>
        <div class="buy" v-show="!food.count || food.count === 0" @click="addFirst($event)" transition="fade">加入购物车</div>
      </div>
      <split v-show="food.info"></split>
      <div class="info" v-show="food.info">
        <h1 class="title">商品介绍</h1>
        <div class="text">{{ food.info }}</div>
      </div>
      <split></split>
      <div class="rating">
        <div class="title">商品评价</div>
        <ratingselect :select-type="selectType" :only-content="onlyContent" :desc="desc"
        :ratings="food.ratings"></ratingselect>
        <div class="rating-wrapper">
          <ul v-if="food.ratings && food.ratings.length">
            <li v-for="rating in food.ratings" v-show="needShow(rating.rateType, rating.text)" class="rating-item">
              <div class="user">
                <span class="name">{{ rating.username }}</span>
                <img :src="rating.avatar"  width="12" height="12" alt="" class="avatar">
              </div>
              <div class="time">{{ rating.rateTime | formatDate }}</div>
              <p class="text">
                <span :class="{'icon-thumb_up': rating.rateType === 0, 'icon-thumb_down': rating.rateType === 1}"></span>
                {{rating.text}}
              </p>
            </li>
          </ul>
          <div class="no-rating" v-else>
            暂无评价
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import cartcontrol from 'components/cartcontrol/cartcontrol'
  import split from 'components/split/split'
  import Vue from 'vue'
  import ratingselect from 'components/ratingselect/ratingselect'
  import { formatDate } from '../../common/js/date'

  // const POSITIVE = 0
  // const NEGATIVE = 1
  const ALL = 2

  export default {
    props: {
      food: {
        type: Object,
        default() {
          return {}
        }
      }
    },
    computed: {
    },
    data() {
      return {
        showFlag: false,
        selectType: ALL,
        onlyContent: true,
        desc: {
          all: '全部',
          positive: '推荐',
          negative: '吐槽'
        }
      }
    },
    methods: {
      show() {
        this.showFlag = true
        //  初始化:因为该组件被商品信息都使用，每当进入新的商品时都希望是初始状态
        this.selectType = ALL
        this.onlyContent = true
        this.$nextTick(() => {
          if (!this.scroll) {
            this.scroll = new BScroll(this.$els.food, {
              click: true
            })
          } else {
            this.scroll.refresh()
          }
        })
      },
      addFirst(event) {
        if (!event._constructed) {
          return
        }
        // 小球动画
        this.$dispatch('cart.add', event.target)
        Vue.set(this.food, 'count', 1)
      },
      needShow(type, text) {
        if (this.onlyContent && !text) {
          return false
        }
        if (this.selectType === ALL) {
          return true
        } else {
          return this.selectType === type
        }
      }
    },
    filters: {
      formatDate(time) {
        let date = new Date(time)
        return formatDate(date, 'yyyy-MM-dd hh:mm')
      }
    },
    components: {
      cartcontrol,
      split,
      ratingselect
    },
    events: {
      'ratingtype.select'(type) {
        this.selectType = type
        // 若没有以下这部分，等数据更新后(比如评论列表增多了)，
        // scroll原先计算的长度并没有变化，所以拖动增多的列表会被反弹回去，
        // 导致无法显式展现在页面上
        // 如果只有refresh()而没有nextTick也会出现问题：
        // 会导致失效，因为vue的数据更新是异步的，refresh执行依然在数据更新前
        this.$nextTick(() => {
          this.scroll.refresh()
        })
      },
      'content.toggle'(onlyContent) {
        this.onlyContent = onlyContent
        this.$nextTick(() => {
          this.scroll.refresh()
        })
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .food
    position: fixed
    left: 0
    top: 0
    bottom: 48px
    z-index: 41
    width: 100%
    background-color: #fff
    &.move-transition
      transition: all 0.2s linear
      transform: translate3d(0,0,0)
    &.move-enter, &.move-leave
      transform: translate3d(100%, 0, 0)
    .image-header
      position: relative
      width: 100%
      height: 0
      padding-top: 100%   // padding-top的百分比是相对于宽度
      img
        position: absolute
        left: 0
        top: 0
        width: 100%
        height: 100%    // absolute的宽高百分比是相对于relative的width+padding总和
      .back
        position: absolute
        top: 5px
        left: 5px
        .icon-arrow_lift
          display: block
          padding: 10px
          font-size: 20px
          color: #fff
          background-color: rgba(7,17,27,0.4)

    .content
      padding: 18px
      border-bottom: 1px solid rgba(7,17,27,0.1)
      position: relative
      .title
        margin: 0 0 8px 0
        font-size: 14px
        color: rgb(7,17,27)
        line-height: 14px
        font-weight: 700
      .detail
        font-size: 0
        margin: 0 0 18px 0
        .sell-count
          margin-right: 12px
          font-size: 10px
          line-height: 10px
          color: rgb(147,153,159)
        .rating
          font-size: 10px
          line-height: 10px
          color: rgb(147,153,159)
      .price
        font-size: 0
        .new
          margin-right: 12px
          font-size: 14px
          line-height: 24px
          font-weight: 700
          color: rgb(240,20,20)
        .old
          text-decoration: line-through
          font-size: 10px
          line-height: 24px
          font-weight: 700
          color: rgb(147,153,159)
      .cartcontrol-warpper
        position: absolute
        right: 12px
        bottom: 12px
      .buy
        position: absolute
        right: 18px
        bottom: 18px
        height: 24px
        line-height: 24px
        padding: 0 12px
        box-sizing: border-box
        font-size: 10px
        border-radius: 12px
        background-color: rgb(0,160,220)
        color: #fff
        &.fade-transition
          transition: all 0.2s
          opacity: 1
        &.fade-enter, &.fade-leave
          opacity: 0



    .info
      padding: 18px
      .title
        margin-bottom: 6px
        font-size: 14px
        line-height: 14px
        color: rgb(7,17,27)
      .text
        padding: 0 8px
        line-height: 24px
        font-size: 12px
        font-weight: 200
        color: rgb(77,85,93)

    .rating
      padding-top: 18px
      .title
        margin-left: 18px
        font-size: 14px
        line-height: 14px
        color: rgb(7,17,27)
      .rating-wrapper
        padding: 0 18px
        .rating-item
          position: relative
          padding: 16px 0
          border-1px(rgba(7,17,27,0.1))
          .user
            position: absolute
            right: 0
            top: 16px
            font-size: 0
            line-height: 12px
            .name
              display: inline-block
              vertical-align: top
              margin-right: 6px
              font-size: 10px
              color: rgb(147,153,159)
            .avatar
              border-radius: 50%


          .time
            margin-bottom: 6px
            line-height: 12px
            font-size: 10px
            color: rgb(147,153,159)

          .text
            line-height: 16px
            font-size: 12px
            color: rgb(7,17,27)
            .icon-thumb_up, .icon-thumb_down
              display: inline-block
              vertical-align: text-top
              line-height: 16px
              margin-right: 4px
              color: rgb(0,160,220)
              font-size: 12px
            .icon-thumb_down
              color: rgb(147,153,159)
      .no-rating
        padding: 16px 0
        font-size: 12px
        color: rgb(147,153,159)
</style>
