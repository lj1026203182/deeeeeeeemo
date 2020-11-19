<template>
  <view class="cart">
    <block v-if="cartList.length">
      <view class="cart-header flex row-between">
        <view>
          当前已选中<text class="text-primary" style="margin: 0 4rpx;">{{tickNum}}</text>件商品(共{{cartList.length}}件)
        </view>
        <view class="cart-heaer-btn" @click="onDel">
          删除
        </view>
      </view>
      <scroll-view scroll-y class="scroll-view">
        <u-gap height="20"></u-gap>

        <view class="cart-swipe" v-for="(item, index) in cartList" :key="item.ID">

          <u-swipe-action :show="item.show" :options="options" :index="index" @click="click" @open="open">
            <view class="cart-box flex row-between">
              <u-checkbox v-model="item.tick" :disabled="item.Stock<=0" size="40" active-color="#E2212A" shape="circle"></u-checkbox>
              <view class="cart-content flex" @click.stop="onDetail(item)">
                <view class="cart-img">
                  <u-image height="100%" border-radius="10" :src="item.Img" mode="aspectFill"></u-image>
                </view>
                <view class="cart-info">
                  <view class="cart-name diandiandian">
                    {{item.ProductName}}
                  </view>
                  <view class="cart-des diandiandian2">
                    {{item.Spec}}
                  </view>
                  <view class="cart-more flex row-between" @click.stop>
                    <view class="cart-price text-price">
                      {{item.SalePrice2}}
                    </view>
                    <u-number-box v-model="item.Amount" :index="index" :disabled="item.Stock<=0" :long-press="false" input-height="60"
                      input-width="60" bg-color="#F7F7F7" :min="1" :max="item.Stock" disabled-input @change="onChangeNum"></u-number-box>
                  </view>
                </view>
              </view>
            </view>
          </u-swipe-action>
        </view>
      </scroll-view>

      <view class="submit-box flex row-between">
        <u-checkbox v-model="isAllTick" active-color="#F3304B" shape="circle">全选</u-checkbox>
        <view class="submit-content flex row-right">
          <view class="submit-nodel">不含运费</view>
          <view class="submit-total">
            合计：
          </view>
          <view class="text-price submit-price">{{$utils.renderFixed(totalPrice)}}</view>
        </view>
        <view class="submit-btn flex row-center" @click="onSubmit">去结算</view>
      </view>
    </block>

    <i-Empty src="/static/empty/cart.png" text="购物车是空的" v-else>
      <view class="empty-btn" @click="$u.route({url:'/pages/category/category',type:'switchTab'})">
        去逛逛
      </view>
    </i-Empty>

    <u-modal v-model="showDel" :show-title="false" :content-style="{color:'#333333'}" confirm-text="删除" async-close
      show-cancel-button confirm-color="#E2212A" content="确定要删除该商品吗？" @confirm="onConfirmDel"></u-modal>

  </view>
</template>

<script>
  import { debounce } from '@/utils/utils.js'
  export default {
    data() {
      return {
        cartList: [],
        options: [{
          text: '删除',
          style: {
            backgroundColor: '#F3304B'
          }
        }],

        showDel: false,
      };
    },
    computed: {
      isAllTick: {
        get() {
          return this.cartList.filter(item => item.Stock > 0).every(item => item.tick)
        },
        set(newVal) {
          this.cartList.forEach(item => { if (item.Stock > 0) item.tick = newVal })
        }
      },
      totalPrice() {
        return this.cartList.filter(item => item.Stock > 0).reduce((total, item) => {
          return total + (item.tick ? (item.Amount * item.SalePrice) : 0)
        }, 0)
      },
      tickNum() {
        return this.cartList.reduce((total, item) => {
          return total + (item.tick ? 1 : 0)
        }, 0)
      }
    },
    onShow() {
      this.getCartList()
    },
    methods: {
      getCartList() {
        this.$api.main.getCartList().then(res => {
          this.cartList = res.data.map(item => ({
            ...item,
            SalePrice2: item.SalePrice.toFixed(2),
            show: false,
            tick: item.Stock > 0
          }))
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      onChangeNum(cartValue) {
        const cart = this.cartList[cartValue.index]
        this.$api.main.cartNum({ ID: cart.ID, ProductID: cart.ProductID, Amount: cartValue.value }).then(res => {
          cart.Amount = cartValue.value
        }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },
      click(index) {
        this.cartList[index].show = false;

        setTimeout(() => {
          this.$api.main.cartDel({ IDS: '' + this.cartList[index].ID }).then(res => {
            this.cartList.splice(index, 1);
          }).catch(err => {
            this.$u.toast(err.msg || '服务器开了个小差~')
          })
        }, 300)
      },
      open(index) {
        this.cartList[index].show = true;
        this.cartList.map((val, idx) => {
          if (index != idx) this.cartList[idx].show = false;
        })
      },
      onDel() {
        if (!this.tickNum) {
          this.$u.toast('请选择要删除的商品~')
          return
        }
        this.showDel = true
      },
      onConfirmDel() {
        this.$api.main.cartDel({ IDS: this.cartList.filter(item => item.tick).map(item => item.ID).toString() }).then(
          res => {
            this.showDel = false;
            this.cartList = this.cartList.filter(item => !item.tick)
          }).catch(err => {
          this.$u.toast(err.msg || '服务器开了个小差~')
        })
      },

      onDetail(item) {
        uni.navigateTo({
          url: `/subpages/shop/shopDetail/shopDetail?id=${item.ProductID}`
        })
      },


      onSubmit: debounce(function() {
        if (!this.tickNum) {
          this.$u.toast('请选择要购买的商品~')
          return
        }

        this.$u.route('/subpages/shop/makeOrder/makeOrder', {
          id: this.cartList.filter(item => item.tick).map(item =>
            item.ID).toString()
        })
      }),
    }
  }
</script>

<style lang="scss">
  page {background-color: #f8f8f8;}
</style>

<style lang="scss" scoped>
  .cart-header {height: 88rpx;background-color: $text-color-fff;padding: 0 32rpx;color: $text-color;font-size: 28rpx;}
  .scroll-view {width: 100%;height: calc(100vh - 98rpx - 88rpx);padding: 0 34rpx;}
  .cart-box {padding: 20rpx;background: #fff;height: 270rpx;width: 676rpx;}
  .cart-swipe {margin-bottom: 20rpx;border-radius: 8rpx;overflow: hidden;}
  .cart::v-deep .u-checkbox__label {margin-right: 0 !important;margin-left: 16rpx !important;}
  .cart-content {flex: 1;min-width: 0;}
  .cart-img {width: 230rpx;height: 230rpx;margin-right: 22rpx;}
  .cart-img image {border-radius: 10rpx;width: 100%;height: 100%;}
  .cart-info {flex: 1;min-width: 0;height: 100%;}
  .cart-name {font-size: 28rpx;color: $text-color;line-height: 44rpx;margin-bottom: 10rpx;font-weight: 600;}
  .cart-des {height: 68rpx;line-height: 34rpx;font-size: 24rpx;color: $text-color9;margin-bottom: 40rpx;}
  .cart-more {}
  .cart-price {align-self: flex-end;line-height: 44rpx;color: $text-price;font-size: 32rpx;font-weight: 600;}
  // 底部
  .submit-box {height: 98rpx;width: 100%;padding: 0 22rpx 0;background-color: $text-color-fff;z-index: 200;position: fixed;bottom: 0;}
  .submit-btn {background-color: $primary-color;color: $text-color-fff;font-weight: 600;font-size: 28rpx;width: 200rpx;height: 78rpx;border-radius: 96rpx;;}
  .submit-content {flex: 1;min-width: 0;margin: 0 40rpx;position: relative;color: $text-color;font-size: 28rpx;}
  .submit-total {position: relative;white-space: nowrap;color: $text-color;}
  .submit-price {color: $text-price;font-weight: 600;}
  .submit-nodel {font-size: 24rpx;color: $text-color9;margin-right: 16rpx;}
  .empty-btn {margin: 60rpx auto 0;width: 294rpx;text-align: center;height: 90rpx;line-height: 90rpx;border-radius: 100rpx;background-color: $primary-color;color: $text-color-fff;font-size: 30rpx;font-weight: 600;}
</style>
