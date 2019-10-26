<template>
  <div class="shopcart">
    <div class="content" @click="toggleList">
      <div class="content-left">
        <div class="logo-wrapper">
          <div class="logo" :class="{ 'highlight': totalCount > 0 }">
            <span class="icon-shopping_cart" :class="{ 'highlight': totalCount > 0 }"></span>
            <div class="num" v-show="totalCount > 0">{{ totalCount }}</div>
          </div>
        </div>
        <div class="price" :class="{ 'highlight': totalPrice > 0 }">￥{{ totalPrice }}</div>
        <div class="desc">另需配送费 ￥{{ deliveryPrice }}元</div>
      </div>
      <div class="content-right" :class="{ 'highlight': totalPrice >= minPrice }">
        <div class="pay" @click.stop.prevent="pay">{{ payDesc }}</div>
      </div>
    </div>
    <div class="ball-container">
      <div transition="drop" v-for="ball in balls" v-show="ball.show"
      class="ball">
        <div class="inner inner-hook"></div>
      </div>
    </div>
    <div class="shopcart-list" v-show="listShow" transition="fold">
      <div class="list-header">
        <div class="title">购物车</div>
        <span class="empty" @click="empty">清空</span>
      </div>
      <div class="list-content" v-el:list-content>
        <ul>
          <li class="food" v-for="food in selectFoods">
            <span class="name">{{ food.name }}</span>
            <div class="price">
              <span>￥{{ food.price * food.count }}</span>
            </div>
            <div class="cartcontrol-wrapper">
              <cartcontrol :food="food"></cartcontrol>
            </div>
          </li>
        </ul>
      </div>
    </div>
  </div>
  <div class="list-mask" @click="fold = true" v-show="listShow" transition="fade"></div>
</template>
<script type="text/ecmascript-6">
  import cartcontrol from '../cartcontrol/cartcontrol'
  import BScroll from 'better-scroll'

  export default {
    props: {
      selectFoods: {
        type: Array,
        default() {
          return []
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        balls: [
          {
            show: false
          },
           {
            show: false
          },
           {
            show: false
          },
           {
            show: false
          },
           {
            show: false
          }
        ],
        dropBall: [],
        fold: true
      }
    },
    computed: {
      totalPrice() {
        console.log('selectFoods', this.selectFoods)
        let total = 0
        this.selectFoods.forEach(item => {
          total += item.price * item.count
        })
        return total
      },
      totalCount() {
        let count = 0
        this.selectFoods.forEach(item => {
          count += item.count
        })
        return count
      },
      payDesc() {
        let str = ''
        if (this.totalPrice === 0) {
          str = `￥${this.minPrice}起送`
        } else if (this.totalPrice >= this.minPrice) {
          str = '去结算'
        } else {
          let tmp = this.minPrice - this.totalPrice
          str = `还差￥${tmp}起送`
        }
        return str
      },
      listShow() {
        if (!this.totalCount) {
          this.fold = true
          return false
        }
        let show = !this.fold
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              this.scroll = new BScroll(this.$els.listContent, {
                click: true
              })
            } else {
              // 重新计算高度来决定是否需要滚动
              this.scroll.refresh()
            }
          })
        }
        return show
      }
    },
    methods: {
      drop(el) {
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i]
          if (!ball.show) {
            ball.show = true
            ball.el = el
            this.dropBall.push(ball)
            return
          }
        }
       },
      toggleList() {
        if (!this.totalCount) {
          return
        }
        this.fold = !this.fold
      },
      empty() {
        this.selectFoods.forEach(food => {
          food.count = 0
        })
      },
      pay() {
        if (this.totalPrice < this.minPrice) {
          return
        }
        window.alert('支付')
      }
    },
    components: {
      cartcontrol
    },
    transitions: {
      drop: {
        beforeEnter(el) {
          let count = this.balls.length
          while (count--) {
            let ball = this.balls[count]
            if (ball.show) {
              let rect = ball.el.getBoundingClientRect()
              let x = rect.left - 32
              let y = -(window.innerHeight - rect.top - 22)
              el.style.display = 'none'
              el.style.webkitTransform = `translate3d(0, ${y}px, 0)`
              el.style.transform = `translate3d(0, ${y}px, 0)`
              let inner = el.getElementsByClassName('inner-hook')[0]
              inner.style.webkitTransform = `translate3d(${x}px, 0, 0)`
              inner.style.transform = `translate3d(${x}px, 0, 0)`
            }
          }
        },
        enter(el) {
          /*eslint-disable no-unused-vars*/
          let rf = el.offsetHeight  // 触发浏览器重绘:对于浏览器重绘参考https://juejin.im/entry/5ab9a4cb51882555677e90ae
          this.$nextTick(() => {
            el.style.webkitTransform = 'translate3d(0, 0, 0)'
            el.style.transform = 'translate3d(0, 0, 0)'
            let inner = el.getElementsByClassName('inner-hook')[0]
            inner.style.webkitTransform = 'translate3d(0, 0, 0)'
            inner.style.transform = 'translate3d(0, 0, 0)'
          })
        },
        afterEnter(el) {
          let ball = this.dropBall.shift()
          if (ball) {
            ball.show = false
            el.style.display = 'none'
          }
        }
      }
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .shopcart
    position: fixed
    left: 0
    bottom: 0
    z-index: 50
    width: 100%
    height: 48px
    .content
      display: flex
      background-color: #141d27
      font-size: 0
      height: 100%
      color: rgba(255,255,255,0.4)
      .content-left
        flex: 1
        .logo-wrapper
          width: 80px
          height: 48px
          display: inline-block
          position: relative
          .logo
            position: absolute;
            width: 56px
            height: 56px
            box-sizing: border-box
            left: 12px
            right: 12px
            bottom: 2px
            border: 6px solid #141d27
            border-radius: 50%
            background-color: #2b343c
            text-align: center
            vertical-align: top
            &.highlight
              background-color: rgb(0,160,220)
              color: #fff
            .icon-shopping_cart
              font-size: 24px
              line-height: 44px
              color: #80858a
              &.highlight
                color: #fff
            .num
              position: absolute
              z-index: 200
              width: 24px
              height: 16px
              right: -6px
              top: -6px
              font-size: 9px
              line-height: 16px
              font-weight: 700
              color: #fff
              background-color: rgb(240,20,20)
              box-shadow: 0 4px 8px 0 rgba(0,0,0,0.4)
              border-radius: 8px
              text-align: center
        .price
          display: inline-block
          vertical-align: top
          margin-top: 12px
          padding-right: 12px
          font-size: 16px
          line-height: 24px
          font-weight: 700
          box-sizing: border-box
          border-right: 1px solid rgba(255,255,255,0.1)
          &.highlight
            color: #fff
        .desc
          display: inline-block
          vertical-align: top
          margin-top: 12px
          padding-left: 12px
          font-size: 12px
          line-height: 24px
      .content-right
        flex: 0 0 105px
        width: 105px
        height: 100%
        background-color: #2b333b
        text-align: center
        &.highlight
          color: #fff
          background-color: #00b43c
        .pay
          display: inline-block
          font-size: 12px
          font-weight: 700
          line-height: 24px
          vertical-align: top
          margin-top: 12px


    .ball-container
      .ball
        position: fixed
        left: 32px
        bottom: 22px
        z-index: 200
        &.drop-transition
          transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41)
          .inner
            width: 16px
            height: 16px
            border-radius: 50%
            background-color: rgb(0,160,220)
            transition: all 0.4s linear
    .shopcart-list
      position: absolute
      left: 0
      top: 0
      width: 100%
      z-index: -1
      &.fold-transition
        transition: all 0.5s
        transform: translate3d(0, -100%, 0)
      &.fold-enter, &.fold-leave
        transform: translate3d(0,0,0)
      .list-header
        height: 40px
        line-height: 40px;
        padding: 0 18px
        background-color: #f3f5f7
        border-bottom: 1px solid rgba(7,17,27,0.1)
        .title
          float: left
          font-size: 12px
          color: rgb(7,17,27)
        .empty
          float: right
          font-size: 12px
          color: rgb(0,160,220)

      .list-content
        padding: 0 12px
        max-height: 217px
        background-color: #fff
        overflow: hidden
        .food
          position: relative
          padding: 12px 0
          box-sizing: border-box
          border-1px(rgba(7,17,27,0.1))
          .name
            line-height: 24px
            font-size: 14px
            color: rgb(7,17,27)
            padding: 0 6px
          .price
            position: absolute
            right: 96px
            bottom: 12px
            line-height: 24px
            font-size: 14px
            font-weight: 700
            color: rgb(240,20,20)
          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 6px

  .list-mask
    position: fixed
    top: 0
    left: 0
    width: 100%
    height: 100%
    z-index: 40
    backdrop-filter: blur(10px)
    &.fade-transition
      transition: all 0.5s
      opacity: 1
      background-color: rgba(7,17,27,0.6)
    &.fade-enter, &.fade-leave
      opacity: 0
      background-color: rgba(7,17,27,0)

</style>
