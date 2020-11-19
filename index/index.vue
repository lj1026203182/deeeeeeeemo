<template>
  <view class="page u-skeleton">
    <!--    <u-navbar :is-back="false" :border-bottom="false" :background="{backgroundColor:backgroundColor}">
      <view class="slot-wrap">
        <view class="u-skeleton-fillet flex1">
          <u-search disabled bg-color="transparent" border-color="#fff" color="#fff" placeholder-color="#fff"
            placeholder="日照香炉生紫烟" :show-action="false"></u-search>
        </view>
      </view>
    </u-navbar> -->
    <u-navbar :is-back="false" :border-bottom="false" :background="{backgroundColor:backgroundColor}">
      <view class="slot-wrap">
        <image class="logo u-skeleton-rect" src="/static/main/logo.png" mode="aspectFit"></image>
      </view>
    </u-navbar>

    <view class="page-wrapper">
      <view class="page-gap" :style="{backgroundColor:backgroundColor}"></view>
      <image class="page-gap-rad" src="/static/main/gap-rad.png"></image>

      <view class="search u-skeleton-fillet">
        <u-search disabled bg-color="transparent" border-color="#fff" color="#fff" placeholder-color="#fff" placeholder="搜索商品-好货等你"
          :show-action="false" @click="$u.route('/subpages/shop/search/search')"></u-search>
      </view>

      <u-swiper class="u-skeleton-fillet" :list="swiperList" name="Img" mode="rect" height="306" border-radius="16"
        @change="onChangeSwiper" @click="onClickSwiper"></u-swiper>

      <!-- icon -->
      <view class="icon-box flex row-around">
        <view class="icon flex-col row-between col-center" v-for="(item,index) in iconList" :key="index" @click="onClickIcon(item)">
          <view class="icon-img-bg flex row-center u-skeleton-circle">
            <image class="icon-img icon-img-active" :src="item.Img" mode="aspectFit"></image>
          </view>
          <view class="icon-text u-skeleton-fillet">{{item.Text}}</view>
        </view>
      </view>

      <!-- banner -->
      <image class="banner u-skeleton-circle" :src="homeImg"></image>

      <!-- 好货推荐 -->
      <view class="good-box" v-if="shopList.length">
        <view class="good-header u-skeleton-rect">
          每日精选好货
        </view>
        <view class="good-content flex flex-wrap">
          <view class="good-card" v-for="(item, index) in shopList" :key="item.ProductID" @click="onClickProduct(item)">
            <view class="good-img u-skeleton-fillet">
              <!-- <image :src="item.Img" mode="aspectFit"></image> -->
              <u-image height="100%" border-radius="10" :src="item.Img" mode="aspectFill"></u-image>
            </view>
            <view class="good-name diandiandian u-skeleton-rect">
              {{item.Name || ''}}
            </view>
            <view class="good-des diandiandian u-skeleton-rect">
              {{item.Intro || ''}}
            </view>
            <view class="good-info">
              <view class="good-info_price text-price u-skeleton-rect">
                {{item.ShowPrice || 0 | toFixed}}
              </view>
              <view class="good-info_count u-skeleton-rect">
                热销 {{item.Sale || ''}} 件
              </view>
            </view>
            <view class="good-more flex row-between">
              <view class="good-more_price text-price u-skeleton-rect">
                {{item.SalePrice || 0 | toFixed}}
              </view>
              <view class="good-more_buy u-skeleton-circle">
                购买
              </view>
            </view>
          </view>
        </view>
      </view>

    </view>

    <u-skeleton :loading="skelonLoading" :animation="true" bgColor="#FFF"></u-skeleton>
  </view>
</template>

<script>
  import { throttle } from '@/utils/utils.js'
  const { windowHeight } = uni.$u.sys()
  let scrollTop = 0
  export default {
    data() {
      return {
        skelonLoading: true,
        backgroundColor: '#fff',
        swiperList: [],
        swiperIndex: 0,
        homeImg: '',
        iconList: [{ Text: "限时抢购", Img: "", Type: 0 }, { Text: "特卖产品", Img: "", Type: 1 }, {
          Text: "TOP10榜",
          Img: "",
          Type: 2
        }, { Text: "更多产品", Img: "", Type: 3 }],
        shopList: [{}, {}, {}, {}]
      }
    },
    onLoad(opt) {
      this.$api.main.getToken().then(res => {
        this.getSwiper()
        this.getIcon()
        this.getHomeProduct()
        this.getHomeImg()
        const ShareUID = this.$utils.getQuery(decodeURIComponent(opt.q), 'ShareUID')
        console.log('%c : ', 'font-size:20px;background-color: #465975;color:#fff;', ShareUID)
        if (ShareUID) uni.setStorageSync('ShareUID', +ShareUID)
      }).catch(err => {
        this.$u.toast(err.msg || '服务器开了个小差~');
      })
    },
    methods: {
      getSwiper() {
        this.$api.main.getSwiper().then(res => {
          this.swiperList = res.data
          if (res.data.length) this.backgroundColor = this.swiperList[0].Color
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      onClickSwiper(index) {
        // Type 0 不跳 1 商品 2 url
        const item = this.swiperList[index]
        if (item.Type === 0) return
        if (item.Type === 1) {
          uni.navigateTo({
            url: `/subpages/shop/shopDetail/shopDetail?id=${item.Url}`
          })
        } else if (item.Type === 2) {
          uni.navigateTo({
            url: `/subpages/shop/webView/webView?Url=${item.Url}`
          })
        }
      },
      onChangeSwiper(index) {
        this.swiperIndex = index
        if (screenTop < windowHeight) this.backgroundColor = this.swiperList[this.swiperIndex].Color
      },

      getIcon() {
        this.$api.main.getIcon().then(res => {
          this.iconList = res.data
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      onClickIcon(item) {
        if (item.Type < 3) this.$u.route('/subpages/shop/iconPage/iconPage', { type: item.Type })
        else this.$u.route({ url: '/pages/category/category', type: 'switchTab' })
      },

      getHomeImg() {
        this.$api.main.getHomeImg({ Pos: 0 }).then(res => {
          this.homeImg = res.data
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },

      getHomeProduct(refresh) {
        this.$api.main.getHomeProduct().then(res => {
          this.shopList = res.data
          this.skelonLoading = false;
          if (refresh === 'refresh') uni.stopPullDownRefresh()
        }).catch(err => {
          this.skelonLoading = false;
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },

      onClickProduct(item) {
        uni.navigateTo({
          url: `/subpages/shop/shopDetail/shopDetail?id=${item.ProductID}`
        })
      },

      onPullDownRefresh() {
        this.getHomeProduct('refresh')
      },

      onPageScroll: throttle(function(e) {
        scrollTop = e.scrollTop
        if (scrollTop > windowHeight) {
          this.backgroundColor = '#F93B42'
        } else {
          this.backgroundColor = this.swiperList[this.swiperIndex].Color
        }
      }, 100),

    }
  }
</script>

<style lang="scss">
  page {background-color: $bg-color2;}
</style>

<style scoped lang="scss">
  .page::v-deep .u-navbar {transition: background-color .3s;}
  .page::v-deep .u-navbar-inner {margin-right: 0 !important;}
  .slot-wrap {display: flex;align-items: center;justify-content: center;flex: 1;padding: 0 30rpx;}
  .logo {width: 180rpx;height: 42rpx;}
  .page-wrapper {padding: 16rpx 32rpx;position: relative;}
  .search {margin-bottom: 36rpx;}
  .page-gap {width: 100%;height: 230rpx;position: absolute;top: 0;left: 0;transition: background-color .3s;z-index: -2;}
  .page-gap-rad {width: 100%;height: 80rpx;position: absolute;top: 150rpx;left: 0;z-index: -1;}
  .icon-box {width: 100%;height: 186rpx;padding: 36rpx 20rpx 36rpx;border-radius: 16rpx;margin: 32rpx 0 48rpx;background-color: $text-color-fff;}
  .icon {flex: 1;height: 128rpx;}
  .icon-img-bg {width: 80rpx;height: 80rpx;margin-bottom: 14rpx;border-radius: 50%;background-color: #fa4f51;}
  .icon-img {width: 60rpx;height: 60rpx;}
  .icon-img-active {animation: icon-scale 2s infinite;}
  .icon-text {font-size: 24rpx;line-height: 34rpx;text-align: center;color: $text-color;}
  .banner {height: 148rpx;margin-bottom: 60rpx;width: 100%;}
  .good-box {border-radius: 8rpx;overflow: hidden;}
  .good-header {height: 76rpx;width: 100%;line-height: 76rpx;padding: 0 30rpx;font-weight: 600;font-size: 32rpx;color: $text-color-fff;background: linear-gradient(270deg, #ffd68d, #f84431);}
  .good-content {background-color: $text-color-fff;padding: 42rpx 8rpx 0rpx 38rpx;width: calc(100% + 8rpx);}
  .good-card {width: 300rpx;margin-right: 26rpx;padding-bottom: 48rpx;}
  .good-card:nth-child(even) {margin-right: 8rpx;}
  .good-img {width: 272rpx;height: 272rpx;margin-bottom: 16rpx;}
  .good-img image {width: 100%;height: 100%;border-radius: 10rpx;background-color: $bg-color2;}
  .good-name {margin-bottom: 4rpx;color: $text-color;font-size: 32rpx;font-weight: 600;line-height: 44rpx;}
  .good-des {color: $text-color9;font-size: 24rpx;line-height: 34rpx;margin-bottom: 8rpx;}
  .good-info {margin-bottom: 8rpx;color: $text-color9;font-size: 24rpx;line-height: 34rpx;}
  .good-info_count {margin-bottom: 12rpx;}
  .good-info_price {text-decoration: line-through;}
  .good-more {}
  .good-more_price {font-size: 36rpx;color: $text-price;font-weight: 600;}
  .good-more_buy {width: 136rpx;height: 60rpx;line-height: 60rpx;text-align: center;color: $text-color-fff;font-size: 28rpx;font-weight: 600;border-radius: 100rpx;background-color: $text-price;}
  @keyframes icon-scale {0% {transform: scale(1)}
    ,
    50% {transform: scale(1.1)}
    ,
    100% {transform: scale(1)}
  }
</style>
