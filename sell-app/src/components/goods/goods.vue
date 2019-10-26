<template>
  <div class="goods">
    <div class="menu-wrapper" v-el:menu-wrapper>
      <ul>
        <li v-for="item in goods" class="menu-item border-1px"
        :class="{ 'current': currentIndex === $index }" @click="selectMenu($index, $event)">
          <span class="text">
            <icon v-show="item.type>0" :size="24" :icon="item.type" :is-full="true"></icon>{{ item.name }}
          </span>
        </li>
      </ul>
    </div>
    <div class="foods-wrapper" v-el:foods-wrapper>
      <ul>
        <li v-for="item in goods" class="food-list food-list-hook">
          <h1 class="title">{{ item.name }}</h1>
          <ul>
            <li v-for="food in item.foods" class="food-item" @click="selectFood(food, $event)">
              <div class="icon">
                <img :src="food.icon" alt="">
              </div>
              <div class="content">
                <h2 class="name">{{ food.name }}</h2>
                <div class="desc">{{ food.description }}</div>
                <div class="extra">
                  <span>月售{{ food.sellCount }}</span>
                  <span>好评率{{ food.rating }}%</span>
                </div>
                <div class="price">
                  <span>￥{{ food.price }}</span>
                  <span v-show="food.oldPrice" class="oldPrice">￥{{ food.oldPrice }}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </div>
            </li>
          </ul>
        </li>
      </ul>
    </div>
    <shopcart v-ref:shopcart :select-foods="selectFoods" :delivery-price="seller.deliveryPrice" :min-price="seller.minPrice"></shopcart>
  </div>
  <food :food="selectedFood" v-ref:food></food>
</template>
<script type="text/ecmascript-6">
  import icon from '../icon/icon'
  import BScroll from 'better-scroll'
  import shopcart from 'components/shopcart/shopcart'
  import cartcontrol from '../cartcontrol/cartcontrol'
  import food from 'components/food/food'

  const ERR_OK = 0

  export default {
    props: {
      seller: {
        type: Object
      }
    },
    created() {
      this.$http.get('api/goods').then(res => {
        res = res.body
        if (res.errno === ERR_OK) {
          this.goods = res.data
          this.$nextTick(() => {
            this._initScroll()
            this._calculateHeight()
          })
        }
      })
    },
    computed: {
      currentIndex() {
        for (let i = 0; i < this.listHeight.length; i++) {
          let height1 = this.listHeight[i]
          let height2 = this.listHeight[i + 1]
          if (!height2 || (this.scrollY >= height1 && this.scrollY <= height2)) {
            return i
          }
        }
      },
      selectFoods() {
        let foods = []
        this.goods.forEach(good => {
          good.foods.forEach(food => {
            if (food.count) {
              foods.push(food)
            }
          })
        })
        return foods
      }
    },
    data() {
      return {
        goods: [],
        listHeight: [],
        scrollY: 0,
        selectedFood: {}
      }
    },
    methods: {
      _initScroll() {
        this.menuScrol = new BScroll(this.$els.menuWrapper, {
          click: true
        })
        this.foodsScroll = new BScroll(this.$els.foodsWrapper, {
          probeType: 3,
          click: true
        })
        this.foodsScroll.on('scroll', (pos) => {
          this.scrollY = Math.abs(Math.round(pos.y))
        })
      },
      _calculateHeight() {
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')
        let height = 0
        this.listHeight.push(height)
        for (let i = 0; i < foodList.length; i++) {
          let item = foodList[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
      },
      selectMenu(index, event) {
        // 避免在PC环境下点击两次（一次是原生点击，一次是better-scroll派发的点击）
        // 原生点击事件是没有_constructed属性，而派发的点击有
        if (!event._constructed) {
          return
        }
        let foodList = this.$els.foodsWrapper.getElementsByClassName('food-list-hook')
        let el = foodList[index]
        this.foodsScroll.scrollToElement(el, 300)
      },
      _drop(target) {
        // 体验优化
        this.$nextTick(() => {
          this.$refs.shopcart.drop(target)
        })
      },
      selectFood(el, event) {
        if (!event._constructed) {
          return
        }
        this.selectedFood = el
        this.$refs.food.show()
      }
    },
    components: {
      icon,
      shopcart,
      cartcontrol,
      food
    },
    events: {
      'cart.add'(target) {
        this._drop(target)
      }
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin"
  .goods
    display: flex
    position: absolute
    width: 100%
    top: 174px
    bottom: 46px
    overflow: hidden
    .menu-wrapper
      flex: 0 0 80px
      width: 80px
      background-color: #f3f5f7
      .menu-item
        display: table
        height: 54px
        width: 56px
        padding: 0 12px
        &.current
          position: relative
          margin-top: -1px
          z-index: 10
          background-color: #fff
          font-weight: 700
          .text
            border-none()
        .text
          display: table-cell
          vertical-align: middle
          font-size: 12px
          font-weight: 400
          line-height: 14px
          border-1px(rgba(7,17,27,0.1))
          .icon
            margin-right: 2px
            vertical-align: top

    .foods-wrapper
      flex: 1
      .title
        padding-left: 14px
        height: 26px
        line-height: 26px
        font-size: 12px
        color: rgb(147,153,159)
        background-color: #f3f5f7
        border-left: 2px solid #d9dde1

      .food-item
        display: flex
        margin: 18px
        padding-bottom: 18px
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none()
        .icon
          flex: 0 0 60px
          margin-right: 10px
          img
            width: 60px
            height: 60px
        .content
          flex: 1
          color: rgb(147,153,159)
          .name
            color: rgb(7,17,27)
            font-size: 14px
            line-height: 14px
            margin-top: 2px
            margin-bottom: 8px
          .desc
            font-size: 10px
            line-height: 12px
            margin-bottom: 8px
          .extra
            line-height: 10px
            span
              margin-right: 12px
              font-size: 10px
          .price
            line-height: 25px
            span:first-child
              color: rgb(240, 20, 20)
              font-size: 14px
              font-weight: 700
              margin-right: 8px
            .oldPrice
              text-decoration: line-through
              color: rgb(147,153,159)
              font-size: 10px
              font-weight: 700


          .cartcontrol-wrapper
            position: absolute
            right: 0
            bottom: 12px
</style>
