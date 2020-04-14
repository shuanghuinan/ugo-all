<template>
  <view class="wrapper">
    <!-- 商品图片 -->
    <swiper class="pics" indicator-dots indicator-color="rgba(255, 255, 255, 0.6)" indicator-active-color="#fff">
      <swiper-item v-for="(item,index) in goodsInfo.pics" :key="index">
        <image :src="item.pics_big"></image>
      </swiper-item>
    </swiper>
    <!-- 基本信息 -->
    <view class="meta">
      <view class="price">￥{{goodsInfo.goods_price}}</view>
      <view class="name">{{goodsInfo.goods_name}}</view>
      <view class="shipment">快递: 免运费</view>
      <text class="collect icon-star">收藏</text>
    </view>
    <!-- 商品详情 -->
    <view class="detail">
      <rich-text v-html="goodsInfo.goods_introduce"></rich-text>
    </view>
    <!-- 操作 -->
    <view class="action">
      <button open-type="contact" class="icon-handset">联系客服</button>
      <text class="cart icon-cart" @click="goCart">购物车</text>
      <text class="add" @click="addToCart">加入购物车</text>
      <text class="buy" @click="createOrder">立即购买</text>
    </view>
  </view>
</template>

<script>
  export default {
    data(){
      return {
        id:null, // 表示当前商品的id值
        goodsInfo:null, // 表示商品信息
        carts:uni.getStorageSync('carts') || [] // 表示购物车信息,本地有就用本地,没有就是[]
      }
    },
    methods: {
      // 跳转到购物车
      goCart () {
        uni.switchTab({
          url: '/pages/cart/index'
        })
      },
      // 立即购买
      createOrder () {
        uni.navigateTo({
          url: '/pages/order/index'
        })
      },
      // 获取商品详情
      async getGoods(){
        let res =await this.http({
          url:"/api/public/v1/goods/detail",
          data:{
            goods_id:this.id
          }
        })
        console.log(res.message);
        this.goodsInfo=res.message        
      },
      // 点击加入购物车时
      addToCart(){
        // 首先要明白一点,购物车数据使用本地存储来管理的
        // 将当前商品信息中的某些信息(与购物车相关的信息), 解析出来存入到一个对象里,这个对象最后要在购物车里面进行渲染
        let { goods_id , goods_name , goods_price , goods_small_logo } = this.goodsInfo
        // 判断当前购物车里面的商品id值有没有和当前商品的id值一样的
        let index = this.carts.findIndex((item)=>{ return item.goods_id === goods_id })

        if(index===-1){
          // 如果没有的话,就将该商品的所有信息加进购物车(找不到就会返回-1,-1表示没找到)
          // ---this_number表示当前商品在购物车中有几份
          // ---还加了一条辅助数据goods_checked,表示该商品在购物车里面是否被选中
          this.carts.push({goods_id , goods_name , goods_price , goods_small_logo , this_number:1 , goods_checked:true})
        }else{
          // 否则, 就将购物车内当前商品的数量加一
          this.carts[index].this_number += 1
        }

        // 然后将购物车数据存入到本地当中
        uni.setStorageSync('carts',this.carts)

        // 提示框
        uni.showToast({title: '加入成功！'});
      }
    },
    onLoad(params){
      this.id=params.id
      // console.log(params);
      this.getGoods()
    }
  }
</script>

<style scoped lang="scss">
  .wrapper {
    margin-bottom: 100rpx;
    background-color: #f4f4f4;
  }

  .pics {
    height: 640rpx;
  }
  
  .meta {
    height: 250rpx;
    line-height: 1;
    padding: 30rpx 180rpx 30rpx 20rpx;
    box-sizing: border-box;
    background-color: #fff;
    position: relative;

    .price {
      font-size: 36rpx;
      color: #ea4451;
      margin-bottom: 20rpx;
    }

    .name {
      color: #333;
      line-height: 1.4;
      font-size: 33rpx;

      display: -webkit-box;
      -webkit-box-orient: vertical;
      -webkit-line-clamp: 2;
      overflow: hidden;
    }

    .shipment {
      font-size: 27rpx;
      color: #999;
      position: absolute;
      bottom: 30rpx;
    }

    .collect {
      width: 140rpx;
      height: 88rpx;
      text-align: center;
      box-sizing: border-box;
      border-left: 1rpx solid #ddd;
      font-size: 24rpx;
      color: #999;

      position: absolute;
      right: 10rpx;
      top: 91rpx;
    }

    [class*="icon-"]::before {
      display: block;
      font-size: 45rpx;
      margin-bottom: 10rpx;
    }
  }

  .detail image {
    width: 100%;
    height: 480rpx;
    margin-top: 20rpx;
  }

  .action {
    width: 100%;
    height: 98rpx;
    background-color: #fff;

    position: fixed;
    left: 0;
    bottom: 0;

    display: flex;
    align-items: center;

    text {
      display: block;
    }

    .add, .buy {
      height: 100%;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 210rpx;
      text-align: center;
      font-size: 27rpx;
      color: #fff;
    }

    .add {
      background-color: #f4b73f;
    }

    .buy {
      background-color: #ea4451;
    }

    button {
      padding: 0;
      border-radius: 0;
      background-color: #fff;

      &::after {
        border: none;
      }
    }

    button, .cart {
      flex: 1;
      text-align: center;
      color: #989898;
      font-size: 24rpx;
      box-sizing: border-box;
    }

    [class*="icon"]::before {
      display: block;
      font-size: 45rpx;
      margin-bottom: 2rpx;
    }
  }
</style>
