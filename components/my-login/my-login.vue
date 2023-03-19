<template>
  <view class="login-container">
    <uni-icons type="contact-filled" size="100" color="#AFAFAF"></uni-icons>
    <!-- <button type="primary" class="btn-login" open-type="getUserInfo" @getuserinfo="getUserInfo">一键登录</button> -->
    <button type="primary" class="btn-login" @click="getUserInfo">一键登录</button>
    <text class="tips-text">登录后尽享更多权益</text>
  </view>
</template>

<script>
  import {
    mapMutations,
    mapState
  } from 'vuex'

  export default {
    data() {
      return {

      };
    },
    computed: {
      ...mapState('m_user', ['redirectInfo'])
    },
    methods: {
      ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
      // 用戶的基本信息
      async getUserInfo() {
        const [err, res] = await uni.getUserProfile({
          desc: '完善信息'
        })
        console.log(res);
        if (err && err.errMsg) return uni.$showMsg('您取消了登录授权')
        this.updateUserInfo(res.userInfo)
        // //判断是否获取用户信息成功

        // // 將用戶的基本信息存儲到vuex中
        // 获取登录成功后的 Token 字符串
        this.getToken(res)
        console.log(this.getToken(res), 22);
      },
      async getToken(info) {
        const [err, res] = await uni.login().catch(err => err)
        // 判斷是否uni.login條用失敗
        if (err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')
        // 準備參數對象
        const query = {
          code: res.code,
          encryptedDate: info.encryptedDate,
          iv: info.iv,
          rawDate: info.rawDate,
          signature: info.signature
        }
        const {
          data: loginResult
        } = await uni.$http.post('/api/public/v1/users/wxlogin', query)
        loginResult.message = {
          token: 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1aWQiOjIzLCJpYXQiOjE1NjQ3MzAwNzksImV4cCI6MTAwMTU2NDczMDA3OH0.YPt-XeLnjV-_1ITaXGY2FhxmCe4NvXuRnRB8OMCfnPo'
        }
        // if(loginResult.meta.status!==200) return uni.$showMsg('登錄失敗')
        uni.$showMsg('登录成功')
        // 2. 更新 vuex 中的 token
        this.updateToken(loginResult.message.token)
        // 判断 vuex 中的 redirectInfo 是否为 null
        // 如果不为 null，则登录成功之后，需要重新导航到对应的页面
        this.navigateBack()
      },
      // 返回登录之前的页面
      navigateBack() {
        // redirectInfo 不为 null，并且导航方式为 switchTab
        if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
          // 调用小程序提供的 uni.switchTab() API 进行页面的导航
          uni.switchTab({
            // 要导航到的页面地址
            url: this.redirectInfo.from,
            // 导航成功之后，把 vuex 中的 redirectInfo 对象重置为 null
            complete: () => {
              this.updateRedirectInfo(null)
            }
          })
        }
      }
    }
    // methods: {
    //   ...mapMutations('m_user', ['updateUserInfo', 'updateToken', 'updateRedirectInfo']),
    //   // 用户授权之后，获取用户的基本信息
    //   getUserInfo(e) {
    //     console.log(e)

    //     if (e.detail.errMsg === 'getUserInfo:fail auth deny') return uni.$showMsg('您取消了登录授权！')

    //     console.log(e.detail.userInfo)
    //     this.updateUserInfo(e.detail.userInfo)

    //     this.getToken(e.detail)
    //   },
    //   async getToken(info) {
    //     // 获取 code 对应的值
    //     const [err, res] = await uni.login().catch(err => err)

    //     if (err || res.errMsg !== 'login:ok') return uni.$showMsg('登录失败！')

    //     // 准备参数
    //     const query = {
    //       code: res.code,
    //       encryptedData: info.encryptedData,
    //       iv: info.iv,
    //       rawData: info.rawData,
    //       signature: info.signature
    //     }

    //     const {
    //       data: loginResult
    //     } = await uni.$http.post('/api/public/v1/users/wxlogin', query)
    //     if (loginResult.meta.status !== 200) return uni.$showMsg('登录失败！')

    //     // 直接把 token 保存到 vuex 中
    //     this.updateToken(loginResult.message.token)
    //     this.navigateBack()
    //   },
    //   navigateBack() {
    //     if (this.redirectInfo && this.redirectInfo.openType === 'switchTab') {
    //       uni.switchTab({
    //         url: this.redirectInfo.from,
    //         complete: () => {
    //           this.updateRedirectInfo(null)
    //         }
    //       })
    //     }
    //   }
    // }
  }
</script>

<style lang="scss">
  .login-container {
    height: 750rpx;
    background-color: #F8F8F8;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    overflow: hidden;

    &::after {
      content: ' ';
      display: block;
      width: 100%;
      height: 40px;
      background-color: white;
      position: absolute;
      bottom: 0;
      left: 0;
      border-radius: 100%;
      transform: translateY(50%);
    }

    .btn-login {
      width: 90%;
      border-radius: 100px;
      margin: 15px 0;
      background-color: #C00000;
    }

    .tips-text {
      font-size: 12px;
      color: gray;
    }
  }
</style>
