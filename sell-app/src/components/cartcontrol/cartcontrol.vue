<template>
  <div class="cartcontrol">
    <span class="cart-decrease icon-remove_circle_outline" v-show="food.count > 0"
    @click.stop.prevent="decreaseCart($event)" transition="move">
      <span class="inner icon-remove_circle_outline"></span>
    </span>
    <span class="cart-count" v-show="food.count > 0">{{ food.count }}</span>
    <span class="cart-add icon-add_circle" @click.stop.prevent="addCart($event)"></span>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue'
  export default {
    props: {
      food: {
        type: Object
      }
    },
    created() {
    },
    methods: {
      addCart(event) {
        if (!event._constructed) return
        if (!this.food.count) {
          // food对象本身里面并没有count属性，因此这种赋值方式definedProper无法检测到，
          // 因为vue不会进行count属性的动态检测
          // this.food.count = 1
          Vue.set(this.food, 'count', 1)
        } else {
          this.food.count++
        }
        this.$dispatch('cart.add', event.target)
      },
      decreaseCart(event) {
        if (!event._constructed) return
        if (this.food.count) {
          this.food.count--
        }
      }
    }
  }
</script>

<style lang="stylus" rel="stylesheet/stylus">
  .cartcontrol
    font-size: 0
    .cart-decrease, .cart-add
      display: inline-block
      padding: 6px
      transition: all 0.4s
      &.move-transition
        opacity: 1
        transform: translate3d(0,0,0)
      .inner
        display: inline-block
        font-size: 24px
        line-height: 24px
        color: rgb(0,160,220)
        transition: all 0.4s linear
        transform: rotate(0)
      &.move-enter, &.move-leave
        opacity: 0
        transform: translate3d(24px, 0, 0)
        .inner
          transform: rotate(180deg)
    .cart-count
      display: inline-block
      vertical-align: top
      width: 12px
      padding: 6px
      line-height: 24px
      text-align: center
      font-size: 10px
      color: rgb(147,153,159)
    .cart-add
      display: inline-block
      font-size: 24px
      line-height: 24px
      padding: 6px
      color: rgb(0,160,220)
</style>
