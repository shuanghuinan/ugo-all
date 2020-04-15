<template>
  <view class="list">
    <!-- 筛选 -->
    <view class="filter">
      <text class="active">综合</text>
      <text>销量</text>
      <text>价格</text>
    </view>
    <!-- 商品列表 -->
    <view class="goods" v-for="item in goods" :key="item.cat_id">
      <view class="item" @click="goDetail(item.cat_id)">
        <!-- 商品图片 -->
        <image class="pic" :src="item.goods_small_logo"></image>
        <!-- 商品信息 -->
        <view class="meta">
          <view class="name">{{item.goods_name}}</view>
          <view class="price">
            <text>￥</text>{{item.goods_price}}<text>.00</text>
          </view>
        </view>
      </view>
    </view>
  <view v-show="isend"> 见底了,没有了 </view>
  </view>
</template>

<script>
  export default {
    data(){
      return {
          parameter:{
            query:null, // 从商品列表接收的参数
            pagenum: 1, //商品页数索引
            pagesize:20 //每页长度
          },
          total:null, //代表一共有多少条商品
          goods:[] ,//用来存放商品列表     
          isend:false // 用来表示列表数据是否全部加载完   
      }
    },
    methods: {
      // 去商品详情页
      goDetail (id) {
        uni.navigateTo({
          url: `/pages/goods/index?id=${id}`
        })
      },
      // 获取商品详情
     async getDetailList(){
        let res=await this.http({
          url:"/api/public/v1/goods/search",
          data:this.parameter
        })
        // console.log(res);
        // 将后台返回的数据加进列表
        this.goods.push(...res.message.goods)
        // 赋值总条数
        this.tatal=res.tatal
      }
    },
    // 下拉加载触发的事件
    onReachBottom(){
        // 如果列表的长度(代表条数)===总条数,则代表到底了没有数据了,应显示"见底了"提示信息,并return回去
        if(this.goods.length===this.total){
          this.isend=true
          return         
        } 
        // 否则就将页码加一,并且发请求
        this.parameter.pagenum += 1
        this.getDetailList()
    },    
    onLoad(params){
      // console.log(params);
      this.parameter.query=params.query
      this.getDetailList()
    }
  }
</script>

<style scoped lang="scss">
  .list{
    padding-top: 100rpx;
  }
  .filter {
    display: flex;
    height: 96rpx;
    line-height: 96rpx;
    border-bottom: 1rpx solid #ddd;

    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    z-index: 9999;
    background-color: #fff;

    text {
      flex: 1;
      text-align: center;
      font-size: 30rpx;
      color: #333;

      &.active {
        color: #ea4451;
      }
    }
  }
  

  .item {
    display: flex;
    padding: 30rpx 20rpx 30rpx 0;
    margin-left: 20rpx;
    border-bottom: 1rpx solid #eee;

    &:last-child {
      border-bottom: none;
    }

    .pic {
      width: 200rpx;
      height: 200rpx;
      margin-right: 30rpx;
    }

    .meta {
      flex: 1;
      font-size: 27rpx;
      color: #333;
      position: relative;
    }

    .name {
      width: 100%;
      overflow: hidden;
      text-overflow: ellipsis;
      display: -webkit-box;
      -webkit-line-clamp: 2;
      -webkit-box-orient: vertical;
    }

    .price {
      position: absolute;
      bottom: 0;

      color: #ea4451;
      font-size: 33rpx;

      text {
        font-size: 22rpx;
      }
    }
  }
</style>
