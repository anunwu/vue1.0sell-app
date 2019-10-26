<template>
  <div class="seller" v-el:seller>
    <div class="seller-content">
      <div class="overview">
        <div class="title">{{seller.name}}</div>
        <div class="desc border-1px">
          <star :size="36" :score="seller.score"></star>
          <div class="text">({{ seller.ratingCount }})</div>
          <div class="text">月售{{ seller.sellCount }}单</div>
        </div>
        <div class="remark">
          <div class="block">
            <h2>起送价</h2>
            <div class="content">
              <span class="stress">{{ seller.minPrice }}</span>元
            </div>
          </div>
          <div class="block">
            <h2>商家配送</h2>
            <div class="content">
              <span class="stress">{{ seller.deliveryPrice }}</span>元
            </div>
          </div>
          <div class="block">
            <h2>平均配送时间</h2>
            <div class="content">
              <span class="stress">{{ seller.deliveryTime }}</span>分钟
            </div>
          </div>
        </div>
        <div class="favorite" @click="toggleFavorite($event)">
          <span class="icon-favorite" :class="{ 'active': favorite }"></span>
          <span class="text">{{ favoriteText }}</span>
        </div>
      </div>
      <split></split>
      <div class="bulletin">
        <div class="title">公告与活动</div>
        <div class="content-wrapper border-1px">
          <p class="content">{{ seller.bulletin }}</p>
        </div>
        <ul v-if="seller.supports" class="supports">
          <li class="support-item border-1px" v-for="item in seller.supports">
            <icon :size="32" :icon="item.type" :is-full="true"></icon>
            <span class="text">{{ item.description }}</span>
          </li>
        </ul>
      </div>
      <split></split>
      <div class="pics">
        <h1 class="title">商家实景</h1>
        <div class="pic-wrapper" v-el:pic-wrapper>
          <ul class="pic-list" v-el:pic-list>
            <li class="pic-item" v-for="pic in seller.pics">
              <img :src="pic" alt="" width="120" height="90">
            </li>
          </ul>
        </div>
      </div>
      <split></split>
      <div class="info">
        <h1 class="title border-1px">商家信息</h1>
        <ul>
          <li class="info-item border-1px" v-for="info in seller.infos">{{ info }}</li>
        </ul>
      </div>
    </div>
  </div>
</template>
<script type="text/ecmascript-6">
  import star from 'components/star/star'
  import split from 'components/split/split'
  import icon from 'components/icon/icon'
  import BScroll from 'better-scroll'
  import { saveToLocal, loadFromLocal } from 'common/js/store'
  export default {
    props: {
      seller: {
        type: Object,
        default() {
          return {}
        }
      }
    },
    watch: {
      // seller是通过请求接口异步更新的，所以在dom第一次更新时seller是空的
      // 也就意味着dom第一次更新时并没有数据撑开盒子，所以此时计算的scroll高度是错误的
      // 当接口请求到seller后，dom产生第二次更新,此时才能获取到seller
      // 只有第二次更新了才能调用_initScroll,此时计算的scroll高度才正确
      'seller'() {
        this._initScroll()
        this._initPics()
      }
    },
    ready() {
      // 当切换到其他组件（如评论），再切回来，实际上全局seller没有变化
      // 而商家组件会重新渲染，但watch观察不到seller数据的变化，所以并不会调用_initScroll
      // 因此在重新渲染组件时也要再调用一次_initScroll
      // this._initScroll()
      // this._initPics()
    },
    data() {
      return {
        favorite: (() => {
          console.log(this.seller.id)
          return loadFromLocal(this.seller.id, 'favorite', false)
        })()
      }
    },
    computed: {
      favoriteText() {
        return this.favorite ? '已收藏' : '收藏'
      }
    },
    methods: {
      toggleFavorite(event) {
        if (!event._constructed) {
          return
        }
        this.favorite = !this.favorite
        saveToLocal(this.seller.id, 'favorite', this.favorite)
      },
      _initScroll() {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$els.seller, {
            click: true
          })
        } else {
          this.scroll.refresh()
        }
      },
      _initPics() {
        // 商家实景scroll
        // pic-list宽度比pic-wrapper盒子宽才能被滚动，但pic-wrapper设置了overflow
        if (this.seller.pics) {
          // 重新设置pic-list的宽度
          let picWidth = 120
          let margin = 6
          let width = (picWidth + margin) * this.seller.pics.length - margin
          this.$els.picList.style.width = width + 'px'
          this.$nextTick(() => {
            if (!this.picScroll) {
              this.picScroll = new BScroll(this.$els.picWrapper, {
                scrollX: true,
                // 忽略垂直方向的滚动(意味着在list-wrapper无法向下滚动)
                eventPassthrough: 'vertical'
              })
            } else {
              this.picScroll.refresh()
            }
          })
        }
      }
    },
    components: {
      star,
      split,
      icon
    }
  }
</script>
<style lang="stylus" rel="stylesheet/stylus">
  @import "../../common/stylus/mixin.styl"
  .seller
    position: absolute
    top: 174px
    bottom: 0
    left: 0
    width: 100%
    overflow: hidden
    .overview
      padding: 18px
      position: relative
      .title
        margin-bottom: 8px
        font-size: 14px
        line-height: 14px
        color: rgb(7,17,27)
      .desc
        font-size: 0
        padding-bottom: 18px
        border-1px(rgba(7,17,27,0.1))
        .star
          display: inline-block
          vertical-align: top
          margin-right: 8px
        .text
          display: inline-block
          vertical-align: top
          line-height: 18px
          margin-right: 12px
          font-size: 10px
          color: rgb(77,85,93)
      .remark
        display: flex
        padding-top: 18px
        .block
          flex: 1
          text-align: center
          border-right: 1px solid rgba(7,17,27,0.1)
          &:last-child
            border: none
          h2
            margin-bottom: 4px
            font-size: 10px
            line-height: 10px
            color: rgb(147,153,159)
          .content
            line-height: 24px
            font-size: 10px
            color: rgb(7,17,27)
            font-weight: 200
            .stress
              font-size: 24px
      .favorite
        position: absolute
        width: 50px
        right: 11px
        top: 18px
        text-align: center
        .icon-favorite
          display: block
          margin-bottom: 4px
          font-size: 24px
          line-height: 24px
          color: #d4d6d9
          &.active
            color: rgb(240,20,20)
        .text
          line-height: 10px
          font-size: 10px
          color: rgb(77,85,93)
    .bulletin
      padding: 18px 18px 0 18px
      .title
        margin-bottom: 8px
        line-height: 14px
        color: rgb(7,17,27)
        font-size: 14px
      .content-wrapper
        padding: 0 12px 16px 12px
        border-1px(rgba(7,17,27,0.1))
        .content
          line-height: 24px
          font-size: 12px
          color: rgb(240,20,20)


      .supports
        .support-item
          padding: 16px 12px 16px 12px
          border-1px(rgba(7,17,27,0.1))
          font-size: 0
          &:last-child
            border-none()
          .icon
            display: inline-block
            vertical-align: top
            margin-right: 6px
          .text
            font-size: 12px
            color: rgb(7,17,27)
            line-height: 16px

    .pics
      padding: 18px
      .title
        margin-bottom: 12px
        line-height: 14px
        color: rgb(7,17,27)
        font-size: 14px
      .pic-wrapper
        width: 100%
        overflow: hidden
        white-space: nowrap
        .pic-list
          font-size: 0
          .pic-item
            display: inline-block
            margin-right: 6px
            width: 120px
            height: 90px
            &:last-child
              margin-right: 0
    .info
      padding: 18px 18px 0 18px
      .title
        padding-bottom: 12px
        line-height: 14px
        border-1px(rgba(7,17,27,0.1))
        color: rgb(7,17,27)
        font-size: 14px
      .info-item
        padding: 16px 12px
        line-height: 16px
        font-size: 12px
        font-weight: 200
        color: rgb(7,17,27)
        border-1px(rgba(7,17,27,0.1))
        &:last-child
          border-none
</style>
