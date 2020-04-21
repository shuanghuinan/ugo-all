<template>
  <view>
    <button type="primary" open-type="getUserInfo" @getuserinfo="getUser" >微信登录</button>
  </view>
</template>

<script>
  export default {
    data(){
      return {
        user:null
      }
    },
    onLoad(){
      uni.login()
    },
    methods:{
      // 按钮标签的开放能力,获取用户信息
      getUser(res){
        // console.log(res)
        // 点击后会出现微信授权页面
        // 如果点击了拒绝,要给一个提示信息. res.detail.errMsg==="getUserInfo:fail auth deny"代表点击了拒绝了
        if(res.detail.errMsg==="getUserInfo:fail auth deny"){
          uni.showToast({ title:"点击拒绝了没办法进行下面的操作哦" , icon:"none"})
          return
        }
        this.user=res.detail
        this.getUserToken()
      },
      // 登录请求--> 登陆后后台就会返回token给我们
      async getUserToken(){
        // 先获取登录所需要的code
        let res=await uni.login()
        let code = res[1].code
        // 发送登录请求
        let result=await this.http({
          url:"/api/public/v1/users/wxlogin",
          method:"post",
          data:{
            code,
            encryptedData:this.user.encryptedData,
            iv:this.user.iv,
            rawData:this.user.rawData,
            signature:this.user.signature
          }
        })
        console.log(result);
        // 如果登录成功
        if(result.meta.status===200){
          uni.setStorageSync("token",result.message.token) // storage存 token
          uni.navigateBack() //返回上一页
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  button {
    width: 600rpx;
    margin: 200rpx auto 0;
  }
</style>