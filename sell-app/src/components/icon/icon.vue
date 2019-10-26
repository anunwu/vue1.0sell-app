<template>
  <span class="icon" :class="[iconType, iconSize]"></span>
</template>

<script>
  export default {
    props: {
      icon: {
        type: Number
      },
      size: {
        type: Number
      },
      isFull: {
        type: Boolean
      }
    },
    computed: {
      iconSize() {
        return 'icon-' + this.size
      },
      iconType() {
        if (this.isFull) {
          return this.classMap[this.icon] + '_1'
        }
        return this.classMap[this.icon]
      }
    },
    data() {
      return {
        classMap: ['decrease', 'discount', 'guarantee', 'invoice', 'special']
      }
    }
  }
</script>

<style lang="stylus" scoped>
  @import "../../common/stylus/mixin"
  .icon
    display: inline-block
    background-repeat: no-repeat
    &.icon-24
      width: 12px
      height: 12px
      background-size: 12px 12px
      &.decrease
        bg-image('decrease_1')
      &.discount
        bg-image('discount_1')
      &.guarantee
        bg-image('guarantee_1')
      &.invoice
        bg-image('invoice_1')
      &.special
        bg-image('special_1')
      &.decrease_1
        bg-image('decrease_3')
      &.discount_1
        bg-image('discount_3')
      &.guarantee_1
        bg-image('guarantee_3')
      &.invoice_1
        bg-image('invoice_3')
      &.special_1
        bg-image('special_3')
    &.icon-32
      width: 16px
      height: 16px
      background-size: 16px 16px
      &.decrease
        bg-image('decrease_2')
      &.discount
        bg-image('discount_2')
      &.guarantee
        bg-image('guarantee_2')
      &.invoice
        bg-image('invoice_2')
      &.special
        bg-image('special_2')
      &.decrease_1
        bg-image('decrease_4')
      &.discount_1
        bg-image('discount_4')
      &.guarantee_1
        bg-image('guarantee_4')
      &.invoice_1
        bg-image('invoice_4')
      &.special_1
        bg-image('special_4')
</style>
