<template>
  <view class="wrapper">
    <!-- 收货信息 -->
    <view class="shipment" v-if="address">
      <view class="dt">收货人: </view>
      <view class="dd meta">
        <text class="name">{{address.userName}}</text>
        <text class="phone">{{address.telNumber}}</text>
      </view>
      <view class="dt">收货地址:</view>
      <view class="dd">{{Address}}</view>
    </view>
    <button v-else type="primary" @click="getAddress">点击添加收货地址</button>
    <!-- 购物车 -->
    <view class="carts">
      <view class="item">
        <!-- 店铺名称 -->
        <view class="shopname">优购生活馆</view>
        <view class="goods" v-for="(item,index) in cartsList" :key="item.goods_id">
          <!-- 商品图片 -->
          <image class="pic" :src="item.goods_small_logo"  @click="toGoods(item.goods_id)"></image>
          <!-- 商品信息 -->
          <view class="meta">
            <view class="name"  @click="toGoods(item.goods_id)">{{item.goods_name}}</view>
            <view class="price">
              <text>￥</text>{{item.goods_price}}<text>.00</text>
            </view>
            <!-- 加减 -->
            <view class="amount">
              <text class="reduce" @click="numSub(index)">-</text>
              <input @input="numChange($event,index)" type="number" :value="item.this_number" class="number">
              <text class="plus" @click="numAdd(index)">+</text>
            </view>
          </view>
          <!-- 选框 -->
          <view class="checkbox">
            <icon @click="isChoose(index)" type="success" size="20" :color="item.goods_checked ? '#ea4451' : '#ccc'"></icon>
          </view>
        </view>
      </view>
    </view>
    <!-- 其它 -->
    <view class="extra">
      <label class="checkall">
        <icon @click="checkAll" type="success" size="20" :color="allGoods ? '#ea4451' : '#ccc'"></icon>
        全选
      </label>
      <view class="total">
        合计: <text>￥</text><label>{{total}}</label><text>.00</text>
      </view>
      <view class="pay" @click="pay">结算({{checkedGoods.length}})</view>
    </view>
  </view>
</template>

<script>
  export default {
    data(){
      return {
        cartsList:[], // 购物车列表
        address:null // 存放地址信息
      }
    },
    methods:{
      // 点击每一项的图片和标题时,去商品详情页
      toGoods(id){
        uni.navigateTo({
           url: `/pages/goods/index?id=${id}`
        });
      },
      // 点加号时,购物车数量加一
      numAdd(index){
        // 当前已加购的数量
        let num=this.cartsList[index].this_number
        // 如果当前已添加的数量>=10,则不能再添加,显示提示信息然后return即可
        if(num>=10){
          uni.showToast({ title:'亲,当前商品最多只能添加10件哦', icon:"none" })
          return
        }else{
          // 否则才可以进行增加操作---但是进行增加操作又有一个前提:有库存
          // 所以又要进行判断 
          if(this.cartsList[index].goods_number>num){
            // 如果当前商品的库存量 > 当前已加购的数量,才可以增加
            this.cartsList[index].this_number+=1
          }else{
            // 否则显示提示信息
            uni.showToast({ title:'亲,当前商品库存已不足了哦,不可以再加了', icon:"none" })
            return
          }
        // 操作完记得存入本地
        uni.setStorageSync('carts', this.cartsList)
        }
      },
      // 点减号时,购物车数量减一
      numSub(index){
        // 当前已加购的数量
        let num=this.cartsList[index].this_number
        // 如果数量<1,就代表不能再减了,给个提示信息并return
        if(num<=1) {
          uni.showToast({ title:'亲,当前商品数量不能再减少了哦', icon:"none" })
          return 
        }
        this.cartsList[index].this_number-=1
        // console.log(num)
        uni.setStorageSync('carts',this.cartsList)
      },
      // 当文本框内的数量变化时
      numChange(e,index){
        // console.log(e)
        // 当前输入框的值   因为value是个字符串,所以要先将其转为数字
        let val = e.detail.value
        if(val==='') return
        val=parseInt(val)
        if( val <= 1){
          val = 1;
        }
        if( val >= 10){
           val = 10;
        }
        // 赋值
        this.cartsList[index].this_number = val
        //存入本地
        uni.setStorageSync('carts', this.cartsList)
      },
      // 点击时切换是否选中
      isChoose(index){
        this.cartsList[index].goods_checked = !this.cartsList[index].goods_checked
        uni.setStorageSync('carts',this.cartsList)
      },
      // 点击全选按钮
      checkAll(){
        // 如果当前所有商品是全选状态,就代表要取消全选
        if(this.allGoods){
          this.cartsList.forEach((item)=>{
          return item.goods_checked=false
          })
        }else{
          // 否则代表要进行全选操作 
          this.cartsList.forEach((item)=>{
            return item.goods_checked=true
          })
        }
        // 存入本地
        uni.setStorageSync('carts',this.cartsList)
      },
      // 点击添加地址按钮
      getAddress(){
        uni.chooseAddress({
          success: (res)=>{
            // console.log(res)
            this.address=res
          }
        });
      },
      // 点击结算按钮
      pay(){
        // 可以结算有三个前提: 1.有收货地址 2.至少选择了一个商品 3.已经登录
        // 如果没有收货地址的话
        if(!this.Address){
          uni.showToast({ title:"选择收货地址之后才可以进行订单的结算哦", icon:"none" })
          return 
        }
        // 如果没有选择商品的话
        if(!this.checkedGoods){
          uni.showToast({ title: '您现在还没有选择商品哦', icon: "none" })
          return
        }
        // 如果没有登录的话
        if(!uni.getStorageSync('token')){
          uni.showToast({ title: '您当前还没有登录哦',icon:"none" })
          uni.navigateTo({ url: "/pages/auth/index"}); // 跳转到登录页面
          return
        }

        // 否则代表满足结算的前提,要跳转到订单页面
      }
       
    },
    computed:{
      // 代表当前已经被选中的商品
      checkedGoods(){
        return this.cartsList.filter((item)=>{
          return item.goods_checked 
        })
      },
      // 代表选中全部商品
      allGoods(){
        return this.cartsList.length===this.checkedGoods.length
      },
      // 当前被选中商品的价格
      total(){
        let total=0
        this.checkedGoods.forEach(item=>{
          return total+=item.goods_price*item.this_number
        })
        return total
      },
      // 收货地址
      Address(){
        if(this.address){
          return this.address.provinceName + this.address.cityName + this.address.countyName + this.address.detailInfo
        }
      }
    },
    onShow(){
      this.cartsList = uni.getStorageSync('carts') || []
      console.log(this.cartsList);
    }

  }
</script>

<style scoped lang="scss">
  .shipment {
    height: 100rpx;
    line-height: 2;
    padding: 30rpx 30rpx 40rpx 30rpx;
    font-size: 27rpx;
    color: #333;
    background-color: #fff;
    background-image: url(http://static.botue.com/ugo/images/cart_border%402x.png);
    background-size: contain;
    background-repeat: no-repeat;
    background-position: bottom;

    .dt {
      width: 140rpx;
      float: left;
      clear: both;
    }

    .dd {
      padding-left: 160rpx;
    }

    .meta {
      padding-right: 50rpx;
    }

    text.phone {
      float: right;
    }
  }

  .carts {
    background-color: #f4f4f4;
    padding-bottom: 110rpx;
    overflow: hidden;

    .shopname {
      padding: 30rpx;
      margin-top: 20rpx;
      font-size: 30rpx;
      color: #333;
      background-color: #fff;
      border-top: 1rpx solid #eee;
      border-bottom: 1rpx solid #eee;
    }

    .goods {
      display: flex;
      padding: 30rpx 20rpx 30rpx 0;
      margin-left: 100rpx;
      border-bottom: 1rpx solid #eee;
      background-color: #fff;
  
      position: relative;

      .checkbox {
        width: 101rpx;
        height: 100%;
        background-color: #fff;

        display: flex;
        justify-content: center;
        align-items: center;

        position: absolute;
        left: -100rpx;
        top: 0;
      }

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

      .amount {
        position: absolute;
        bottom: 0;
        right: 20rpx;

        height: 48rpx;
        text-align: center;
        border: 1rpx solid #ddd;
        border-radius: 8rpx;

        display: flex;
        align-items: center;

        text {
          display: block;
          width: 60rpx;
          line-height: 48rpx;
          font-size: 36rpx;
          color: #ddd;
          text-align: center;
        }

        input {
          width: 60rpx;
          height: 48rpx;
          min-height: 48rpx;
          font-size: 27rpx;
          border-left: 1rpx solid #ddd;
          border-right: 1rpx solid #ddd;
        }
      }
    }
  }

  .extra {
    position: fixed;
    bottom: 0;
    /* #ifdef H5 */
    bottom: 50px;
    /* #endif */
    left: 0;
    z-index: 9;

    width: 750rpx;
    height: 96rpx;
    text-align: center;
    line-height: 96rpx;
    font-size: 36rpx;
    border-top: 1rpx solid #eee;
    background-color: #fff;
    color: #333;
    display: flex;

    .checkall {
      width: 140rpx;
      line-height: 1;
      margin-left: 25rpx;
      display: flex;
      align-items: center;

      icon {
        margin-right: 20rpx;
      }
    }

    .total {
      display: flex;
      justify-content: center;
      flex: 1;

      label, text {
        color: #ea4451;
        vertical-align: bottom;
        position: relative;
        bottom: -2rpx;
      }

      text {
        position: relative;
        bottom: -3rpx;
        font-size: 24rpx;

        &:first-child {
          margin-left: 10rpx;
        }
      }
    }

    .pay {
      width: 200rpx;
      background-color: #ea4451;
      color: #fff;
    }
  }
</style>

