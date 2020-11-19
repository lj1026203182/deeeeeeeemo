<template>
  <view class="u-wrap">
    <view class="u-search-box solid-bottom" @click="$u.route('/subpages/shop/search/search')">
      <view class="u-search-inner">
        <u-icon name="search" color="#909399" :size="28"></u-icon>
        <text class="u-search-text">搜索商品-好货等你</text>
      </view>
    </view>
    <view class="u-menu-wrap">
      <scroll-view scroll-y scroll-with-animation class="u-tab-view menu-scroll-view" :scroll-top="scrollTop">
        <view v-for="(item,index) in tabbar" :key="index" class="u-tab-item" :class="[current==index ? 'u-tab-item-active' : '']"
          @tap.stop="swichMenu(index)">
          {{item.Name}}
        </view>
      </scroll-view>
      <block>
        <scroll-view scroll-y class="right-box" @scrolltolower="getCategoryList('more')">
          <view class="shop-card flex row-between" v-for="(it, i) in tabbar[current].list" :key="it.ProductID" @click="onDetail(it)">
            <view class="shop-img">
              <!-- <image :src="it.Img"></image> -->
              <u-image height="100%" border-radius="10" :src="it.Img" mode="aspectFill"></u-image>
            </view>
            <view class="shop-content solid-bottom">
              <view class="shop-name diandiandian">
                {{it.Name}}
              </view>
              <view class="shop-des diandiandian">
                {{it.Intro}}
              </view>
              <view class="shop-info">
                已售 {{it.Sale}} 件
              </view>
              <view class="shop-more flex row-between">
                <view class="shop-more_price flex-col row-between">
                  <view class="text-price shop-more_price-main">
                    {{it.SalePrice | toFixed}}
                  </view>
                  <view class="shop-more_price-sale">
                    ￥{{it.ShowPrice | toFixed}}
                  </view>
                </view>
                <view class="shop-more_buy">
                  购买
                </view>
              </view>
            </view>
          </view>
        </scroll-view>
      </block>
    </view>
  </view>
</template>

<script>
  export default {
    data() {
      return {
        tabbar: [{ Name: '', list: [] }],
        page: 1,

        scrollTop: 0, //tab标题的滚动条位置
        current: 0, // 预设当前项的值
        menuHeight: 0, // 左边菜单的高度
        menuItemHeight: 0, // 左边菜单item的高度
      }
    },
    computed: {

    },
    onLoad() {
      this.getCategoryType()
    },
    methods: {
      getCategoryType() {
        this.$api.main.getCategoryType().then(res => {
          this.tabbar = res.data.map(item => ({ ...item, list: [] }))
          this.getCategoryList()
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      getCategoryList(more) {
        const type = this.tabbar[this.current]
        this.$api.main.getCategoryList({ TypeID: type.ID, Page: this.page }).then(res => {
          if (this.page === 1) type.list = res.data
          else type.list = type.list.concat(res.data)

          if (res.data.length) this.page += 1
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },

      onDetail(item) {
        uni.navigateTo({
          url: `/subpages/shop/shopDetail/shopDetail?id=${item.ProductID}`
        })
      },

      // 点击左边的栏目切换
      swichMenu(index) {
        if (index == this.current) return;
        this.current = index;
        // 如果为0，意味着尚未初始化
        if (this.menuHeight == 0 || this.menuItemHeight == 0) {
          this.getElRect('menu-scroll-view', 'menuHeight');
          this.getElRect('u-tab-item', 'menuItemHeight');
        }
        // 将菜单菜单活动item垂直居中
        this.scrollTop = index * this.menuItemHeight + this.menuItemHeight / 2 - this.menuHeight / 2;

        this.page = 1
        this.getCategoryList()
      },
      // 获取一个目标元素的高度
      getElRect(elClass, dataVal) {
        const query = uni.createSelectorQuery().in(this);
        query.select('.' + elClass).fields({ size: true }, res => {
          // 如果节点尚未生成，res值为null，循环调用执行
          if (!res) {
            setTimeout(() => {
              this.getElRect(elClass, dataVal);
            }, 10);
            return;
          }
          this[dataVal] = res.height;
        }).exec();
      }
    }
  }
</script>

<style lang="scss" scoped>
  .u-wrap {height: calc(100vh);
    /* #ifdef H5 */
    height: calc(100vh - var(--window-top));
    /* #endif */
    display: flex;flex-direction: column;}
  .u-search-box {padding: 20rpx 34rpx;}
  .u-menu-wrap {flex: 1;display: flex;overflow: hidden;}
  .u-search-inner {background-color: #F7F7F7;border-radius: 100rpx;border: 1rpx solid #e1e1e1;height: 60rpx;display: flex;align-items: center;padding: 0 28rpx;}
  .u-search-text {font-size: 26rpx;color: $u-tips-color;margin-left: 10rpx;}
  .u-tab-view {width: 156rpx;height: 100%;}
  .u-tab-item {height: 80rpx;background: #f6f6f6;box-sizing: border-box;display: flex;align-items: center;justify-content: center;font-size: 24rpx;color: #2D2B2B;font-weight: 400;line-height: 1.4;text-align: center;word-break: break-all;}
  .u-tab-item-active {position: relative;height: 94rpx;color: $primary-color;font-size: 28rpx;font-weight: 600;background: #fff;transition: .1s;}
  .right-box {flex: 1;min-width: 0;}
  .shop-card {padding: 28rpx 34rpx 0 16rpx;}
  .shop-img {width: 200rpx;height: 200rpx;margin-right: 18rpx;align-self: flex-start;}
  .shop-img image {width: 100%;height: 100%;border-radius: 10rpx;background: $bg-color2;}
  .shop-content {flex: 1;min-width: 0;height: 222rpx;}
  .shop-name {color: $text-color;font-size: 28rpx;font-weight: 600;margin-bottom: 2rpx;line-height: 40rpx;}
  .shop-des {color: $text-color9;font-size: 24rpx;line-height: 34rpx;margin-bottom: 8rpx;}
  .shop-info {color: $text-color9;font-size: 24rpx;line-height: 34rpx;margin-bottom: 4rpx;}
  .shop-more {}
  .shop-more_price {align-self: flex-end;flex: 1;min-width: 0;}
  .shop-more_price-main {color: $text-price;font-size: 32rpx;font-weight: 600;line-height: 44rpx;}
  .shop-more_price-sale {color: $text-color9;font-size: 24rpx;text-decoration: line-through;line-height: 34rpx;}
  .shop-more_buy {height: 52rpx;width: 116rpx;align-self: flex-end;line-height: 52rpx;text-align: center;font-size: 28rpx;font-weight: 600;color: $text-color-fff;border-radius: 100rpx;background-color: $primary-color;}
</style>
