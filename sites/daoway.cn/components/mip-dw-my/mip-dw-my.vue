<template>
  <div class="wrapper">
    <div class="mybg" >
      <img
        :src="userInfo.iconUrl?userInfo.iconUrl:'http://www.daoway.cn/images/myicon.png'"
        class="myicon">
      <div
        v-if="userId"
        class="txt">
        <div class="username">{{ userInfo.nick }}
          <img
            :src="userInfo.levelIcon"
            class="starimg">
        </div>
      </div>
      <div
        v-if="!userId"
        class="txt2"
        @click="goLoginPage">请登录</div>
    </div>
    <div
      v-if="userInfo.couponCount"
      class="mylist"
      style="margin-top:0; border:0"
      @click="goVouchersPage" >
      <div class="item">
        <img
          src="http://www.daoway.cn/images/quan.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">代金券</div>
        <div class="carttel">{{ userInfo.couponCount }}张<img
          class="m-more"
          src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div class="mylist">
      <div class="item" >
        <img
          src="http://www.daoway.cn/images/tel.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">联系客服</div>
        <div class="carttel"><a
          href="tel:400-0908-608"
          rel="external">400-0908-608</a><img
            class="m-more"
            src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div class="mylist">
      <div
        class="item"
        @click="toabout">
        <img
          src="http://www.daoway.cn/images/about.png"
          class="cartimg"
          style="width:20px; height:20px">
        <div class="cart">关于</div>
        <div class="carttel"><img
          class="m-more"
          src="https://www.daoway.cn/h5/image/go_06.png"></div>
      </div>
    </div>
    <div
      class="exit"
      @click="loginOut">退出登录</div>

    <!--提示-->
    <div
      v-show="warn.show"
      class="layer">
      <div class="layer-content zoomIn">
        <p
          class="layer-text"
          v-text="warn.texts"/>
        <p
          class="layer-sure active-layer"
          @click="closeLayer">知道了</p>
      </div>
    </div>
  </div>
</template>
<script>
import base from '../../common/utils/base'
/* import login from '../../common/utils/login' */
import '../../common/utils/base.less'

export default {
  data () {
    return {
      phoneNumber: '400-0908-608',
      iconUrl: 'http://www.daoway.cn/mip/common/images/myicon.png',
      couponCount: 0,
      userInfo: {},
      warn: {
        // 弹窗
        show: false,
        texts: ''
      },
      redirect_uri: 'http://test.daoway.cn/mip/t/my.html',
      client_id: 'vnQZ7pPB0gsWHZZF4n6h0WDOl8KOr7Lq',
      ClientSecret: 'kM6rbBN43zhAEOFxeQ9Wnj2MzVzkROA0',
      code: base.getRequest(location.href).code,
      userId: ''
    }
  },
  mounted () {
    let userId = localStorage.getItem('userId')
    let token = localStorage.getItem('token')
    if (!userId && !token) {
      MIP.viewer.open(base.htmlhref.index, { isMipLink: true })
    } else {
      this.userId = userId
      this.token = token
      document.cookie = 'token=' + token + ';path=/'
      this.getmyhtml()
    }
  },
  methods: {
    showloading () {
      this.loading = true
    },
    closeLayer () {
      this.warn.show = false
    },
    getmyhtml () {
      let that = this
      let url = '/daoway/rest/user/' + that.userId + '?isowner=1'
      fetch(url, {
        method: 'get',
        credentials: 'include',
        headers: {
          'content-type': 'application/x-www-form-urlencoded',
          'cookie': 'token=' + this.token
        }
      }).then(function (res) {
        return res.json()
      }).then(function (text) {
        console.log(text)
        if (text.status === 'ok') {
          let data = text.data
          let userInfo = {}
          if (data.iconUrl) {
            userInfo.iconUrl = data.iconUrl ? data.iconUrl : 'http://www.daoway.cn/images/myicon.png'
          }
          userInfo.nick = data.nick
          userInfo.couponCount = data.couponCount
          let level = data.level
          if (level === '100') { // 青铜会员
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/q.png'
            userInfo.levelhtml = '青铜会员'
          } else if (level === '200') { // 白银会员
            userInfo.levelhtml = '白银会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/b.png'
          } else if (level === '300') { // 黄金会员
            userInfo.levelhtml = '黄金会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/h.png'
          } else if (level === '400') { // 钻石会员
            userInfo.levelhtml = '钻石会员'
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/imagesz.png'
          } else {
            userInfo.levelIcon = 'http://www.daoway.cn/mip/common/images/p.png'
          }
          that.userInfo = userInfo
        } else {
          that.warn.show = true
          that.warn.texts = text.msg
        }
      }).catch(function (error) {
        console.log(error)
      })
    },
    loginOut: function () {
      let that = this
      that.userId = null
      that.userInfo.iconUrl = 'http://www.daoway.cn/images/myicon.png'
      that.userInfo.couponCount = 0
    },
    goLoginPage: function () {
      let that = this
      let url = 'https://openapi.baidu.com/oauth/2.0/authorize?response_type=code&client_id=' + that.client_id + '&redirect_uri=' + that.redirect_uri + '&scope=snsapi_userinfo&state=STATE'
      MIP.viewer.open(url)
    },
    goVouchersPage: function () {
      let miniUserId = this.userId
      if (miniUserId) {
        MIP.viewer.open(base.htmlhref.vouchers, { isMipLink: true })
      } else {
        this.userId = ''
      }
    },
    toabout () {
      MIP.viewer.open(base.htmlhref.about, { isMipLink: true })
    }
  }
}
</script>
<style scoped>
    * {
        padding: 0;
        margin: 0;
        box-sizing: border-box;
    }

    i {
        font-style: normal
    }

    li, ol {
        list-style: none
    }
    .regclolr{
        color:#f64e4e ;
    }

    .layer-text{
        text-align: center;
    }

    .mybg {
        width: 700px;
        height: 134px;
        position: relative;
        padding: 35px 15px;
        color: #fff;
        background: -webkit-linear-gradient(#fd5f46, #fd322b);
        background: linear-gradient(#fd5f46, #fd322b);

    }

    .username, .txt, .starimg, .myicon, .set, .rightarrow {
        display: inline-block;
    }
    .username{
        font-size: 18px;
    }

    .myicon {
        width: 64px;
        height: 64px;
        border-radius: 100%;
        border: 3px solid #fff;
        margin-left: 10px;
    }

    .txt {
        width: 60%;
        line-height: 30px;
        vertical-align: top;
        margin-top: 15px;
        margin-left: 10px;
    }

    .txt2 {
        width: 20%;
        line-height: 30px;
        vertical-align: top;
        margin-top: 15px;
        margin-left: 10px;
        display: inline-block;
    }

    .starimg {
        width: 17px;
        height: 17px;
        position: relative;
        top: 2px;
        margin-right: 3px;
    }

    .mylist {
        width: 100%;
        background: #fff;
        margin-top: 10px;
    }

    .cart {
        display: inline-block;
        color: #4c4c4c;
        font-size: 16px
    }

    .cartimg {
        position: absolute;
        top: 12px;
        left: 1.5%;
    }

    .item {
        margin-left: 1.5%;
        width: 97%;
        height: 40px;
        line-height: 40px;
        position: relative;
    }

    .cart {
        margin-left: 10%;
    }

    .carttel {
        float: right;
        margin-right: 3%;
        color: #898989;
        font-size: 14px
    }

    .exit {
        text-align: center;
        width: 94%;
        margin: 0 3%;;
        border: none;
        border-radius: 5px;
        height: 40px;
        line-height: 40px;
        background: #fff;
        margin-top: 20px;
        color: #4c4c4c;
        font-size: 16px
    }
    .m-more{
        width: 7px;
        height: auto;
        margin-left: 5px;
    }

</style>
