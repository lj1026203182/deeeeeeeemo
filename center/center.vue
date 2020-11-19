<template>
  <view>
    <u-navbar :is-back="false" :border-bottom="false" title="我的" title-color="#fff" :background="{background:'#fb233a'}"></u-navbar>
    <image class="banner" src="/static/main/banner.png"></image>
    <view class="wraper">
      <view class="user">
        <i-UserBtn :applyDetail="applyDetail"></i-UserBtn>
      </view>
      <view class="order">
        <view class="title">
          我的订单
        </view>
        <view class="order-content flex row-between flex-wrap">
          <view class="order-item flex-col col-center" v-for="(item, index) in iconList" :key="item.id" @click="onIcon(item)">
            <image class="order-img" :src="`/static/main/center${item.id}.png`"></image>
            <view class="order-text">{{item.title}}</view>
            <u-badge bgColor="#F93B42" :count="item.num" :offset="[-40,-4]"></u-badge>
          </view>
        </view>
      </view>

      <view class="notice" v-if="list.length && userInfo.IsAgent">
        <u-notice-bar :list="list" more-icon bg-color="#f7f7f7" color="#FF9D00" padding="0"></u-notice-bar>
      </view>

      <view class="activity flex row-between" v-if="userInfo.IsAgent">
        <image src="/static/main/center-ban-0.png" @click="$u.route('/subpages/shop/income/income')"></image>
        <image src="/static/main/center-ban-1.png" @click="$u.route('/subpages/shop/qrCode/qrCode')"></image>
      </view>

      <view class="server">
        <view class="title">
          我的订单
        </view>
        <view class="server-content flex flex-wrap">
          <view class="server-item flex-col col-center" v-for="(item, index) in serverListMain" :key="item.id" @click="onServer(item)">
            <button v-if="item.id === 10" class="u-reset-button server-item-btn flex-col col-center" open-type="contact">
              <image class="server-img" :src="`/static/main/server${item.id}.png`"></image>
              <view class="server-text">{{item.title}}</view>
            </button>
            <block v-else>
              <image class="server-img" :src="`/static/main/server${item.id}.png`"></image>
              <view class="server-text">{{item.title}}</view>
            </block>
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        iconList: [{
          title: '待付款',
          id: 0,
          num: 0
        }, {
          title: '待发货',
          id: 1,
          num: 0
        }, {
          title: '待收货',
          id: 2,
          num: 0
        }, {
          title: '已收货',
          id: 3,
          num: 0
        }, {
          title: '已退单',
          id: 4,
          num: 0
        }, {
          title: '已售后',
          id: 5,
          num: 0
        }],
        serverListVip: [{
          title: '我的代理',
          id: 6,
          url: '/subpages/shop/agent/agent',
        }, {
          title: '我的销量',
          id: 7,
          url: '/subpages/shop/sales/sales',
        }, {
          title: '素材圈',
          id: 8,
          url: '/subpages/shop/sourceArea/sourceArea',
        }],
        serverList: [{
          title: '地址管理',
          id: 9,
          url: '/subpages/shop/address/address'
        }, {
          title: '联系客服',
          id: 10,
        }, {
          title: '消息设置',
          id: 11,
        }],
        list: [],
        applyDetail: {}
      };
    },
    computed: {
      userInfo() {
        return this.$store.state.user.userInfo
      },
      serverListMain() {
        return this.userInfo.IsAgent === 1 ? [...this.serverListVip, ...this.serverList] : this.serverList
      }
    },
    onShow() {
      if (this.userInfo.IsAgent === 1) this.getLastNotice()
      else this.isApplyAgent()
    },
    methods: {
      onIcon(item) {
        this.$u.route('/subpages/shop/order/order', { type: item.id })
      },
      onServer(item) {
        // 非消息设置
        if (item.id !== 11) this.$u.route(item.url)
        else {
          this.$api.main.subMessage(['M0AkhMaJ7PR81hgFcB6eL8jwB8qQOo2WdiFfQyP9FDM',
            'NL9TFlHO5LkiQ5y4O0Wzq_lZl5O3gfJIFzvpgWSTTM4'
          ])
        }
      },
      getLastNotice() {
        this.$api.main.getLastNotice().then(res => {
          if (res.data) this.list = [res.data.Title]
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      isApplyAgent() {
        this.$api.main.isApplyAgent().then(res => {
          this.applyDetail = res.data
        })
      }
    }
  }
</script>

<style lang="scss">
  page {background-color: $bg-color2;}
</style>
<style lang="scss" scoped>
  .banner {width: 100%;height: 260rpx;z-index: -1;position: relative;}
  .wraper {padding: 0 32rpx;margin-top: -242rpx;}
  .user {margin-bottom: 40rpx;}
  .title {color: $text-color;font-size: 32rpx;font-weight: 600;line-height: 44rpx;}
  .order {background-color: $text-color-fff;border-radius: 8rpx;padding: 22rpx 32rpx 40rpx;overflow: hidden;margin-bottom: 30rpx;}
  .order-content {width: calc(100% + 30rpx);margin-left: -30rpx;}
  .order-item {width: 33.33%;position: relative;margin: 28rpx 0 0;}
  .order-img {width: 56rpx;height: 56rpx;margin-bottom: 14rpx;}
  .order-text {text-align: center;font-size: 24rpx;line-height: 34rpx;color: $text-color;}
  .order-brage {position: absolute;right: 0;top: -32rpx;}
  // notice
  .notice {margin-bottom: 36rpx;}
  // activity
  .activity {width: 100%;height: 160rpx;margin-bottom: 32rpx;}
  .activity image {width: 336rpx;height: 100%;border-radius: 8rpx;}
  // 服务帮助
  .server {background-color: $text-color-fff;border-radius: 8rpx;padding: 32rpx 32rpx 15rpx;overflow: hidden;margin-bottom: 32rpx;}
  .server-content {padding-top: 22rpx;}
  .server-item {width: 25%;margin-bottom: 44rpx;}
  .server-item-btn {width: 100%;}
  .server-img {width: 100rpx;height: 100rpx;}
  .server-text {text-align: center;font-size: 24rpx;line-height: 34rpx;color: $text-color;}
</style>
