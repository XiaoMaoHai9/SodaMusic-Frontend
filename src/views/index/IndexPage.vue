<template>
  <div class="html-container no-scroll">
    <!-- 顶部导航栏 -->
    <div class='banner'>
      <ul class='top-banner'>
        <li>
          <img class="web-logo" src="@/assets/images/logos/sodamusic.png" alt="logo" @click="$router.push('/found')">
          <div class="top-nav-container" ref="topNav">
            <span class="top-nav" ref="topNavFound" @click="$router.push('/found')">发现音乐</span>
            <span class="top-nav" ref="topNavLib" @click="$router.push('/myMicLib')">私人乐库</span>
          </div>
        </li>
        <li>
          <search-bar></search-bar>
          <a-button type="primary" size="large" :style="{ width: '80px', fontSize: '15px', verticalAlign: 'middle'}" @click="loginFlag = true" v-if="!thirdParty.isLogin && !sodaAccount.isLogin">登录</a-button>
          <a-popover>
            <template slot="content">
              <ul class="bannner-popover-ul">
                <li class="br" @click="loginFlag = true">
                  <span>
                    <a-icon type="api" :style="{ marginRight: '7px'}"/>切换接口</span>
                </li>
                <li class="br" @click="handleLogout">
                  <span>
                    <a-icon type="logout" :style="{ marginRight: '7px'}"/>退出登录</span>
                </li>
              </ul>
            </template>
            <span class="avatar-container" @click="clickAvatar">
              <a-avatar class="tran-03s" style="z-index: 2" :size="40" :src="thirdParty.userInfo.avatarUrl || sodaAccount.userInfo.avatar_url" v-if="thirdParty.isLogin || sodaAccount.isLogin"/>
            <a-avatar class="tran-03s" style="left: -25px; z-index: 1" :size="40" :src="sodaAccount.userInfo.avatar_url" v-if="thirdParty.isLogin && sodaAccount.isLogin"/>
            </span>
          </a-popover>
        </li>
      </ul>
    </div>
    <!-- 主内容区 -> 路由跳转 -->
    <div class='main-container'>
      <!-- 组件缓存 -->
      <keep-alive :include="keepArr">
        <router-view/>
      </keep-alive>
    </div>
    <!-- 页脚 -->
    <div class='footer'>
      <div class="footer-content">
          <div class="songs-link footer-info">
            <ul>
              <li class="info-title">歌源链接</li>
              <li>
                <a class="info-link" href="https://music.163.com/">网易云音乐
                  <img src="@/assets/images/logos/neteasecloud-icon.svg" alt="">
                </a>
              </li>
              <li>
                <a class="info-link" href="https://y.qq.com/">QQ音乐
                  <img src="@/assets/images/logos/QQmusic-icon.svg" alt="">
                </a>
              </li>
              <li>
                <a class="info-link" href="https://www.kugou.com/">酷狗音乐
                  <img src="@/assets/images/logos/kugoumusic-icon.svg" alt="">
                </a>
              </li>
            </ul>
          </div>
          <div class="my-link footer-info">
             <ul>
              <li class="info-title">与我联系</li>
              <li>
                <a class="info-link"  href="https://github.com/XiaoMaoHai9">Github
                  <img src="@/assets/images/logos/Github-icon.svg" alt="">
                </a>
              </li>
              <li>
                <a class="info-link" href="https://space.bilibili.com/66681654?spm_id_from=333.337.0.0">Blibili
                  <img src="@/assets/images/logos/Bilibili-icon.svg" alt="">
                </a>
              </li>
              <li>
                <a class="info-link" href="https://weibo.com/u/5125621060">Weibo
                  <img src="@/assets/images/logos/Weibo-icon.svg" alt="">
                </a>
              </li>
            </ul>
          </div>
          <div class="other-link footer-info">
            <ul>
              <li><a class="info-link">©  2024 小毛孩</a></li>
              <li><a class="info-link">版权声名</a></li>
              <li><a class="info-link">捐助我</a></li>
            </ul>
          </div>
      </div>
    </div>
    <!-- 登录卡片 -->
    <transition name="mask-fade">
      <login-window v-if="loginFlag" @closeWindow="closeWindow()" @goRegister="goNewWindow('register')"></login-window>
    </transition>
    <!-- 注册卡片 -->
    <transition name="mask-fade">
      <register-card v-if="registerFlag" @closeWindow="closeWindow()" @goLogin="goNewWindow('login')"></register-card>
    </transition>
    <!-- 音频播放器 -->
    <audio-player  v-if="audioPlayer.isStart"></audio-player>
    <!-- 视频播放器 -->
    <transition name="mask-fade">
      <video-player v-if="videoPlayer.isStart"></video-player>
    </transition>
    <!-- 模糊背景蒙版 -->
    <div class="win-bk-mask mask-style-b" v-if="loginFlag || registerFlag || videoPlayer.isStart" @click="closeWindow()">
    </div>
  </div>
</template>

<script>
import { mapMutations, mapState } from 'vuex'
import LoginWindow from '@/components/common/LoginCard.vue'
import RegisterCard from '@/components/common/RegisterCard.vue'
import AudioPlayer from '@/components/common/AudioPlayer.vue'
import VideoPlayer from '@/components/common/VideoPlayer.vue'
import SearchBar from '@/components/common/SearchBar.vue'

export default {
  name: 'IndexPage',
  components: {
    LoginWindow,
    AudioPlayer,
    VideoPlayer,
    SearchBar,
    RegisterCard
  },
  data () {
    return {
      keepArr: ['FoundMusic', 'MusicLibPage'],
      loginFlag: false,
      registerFlag: false,
      avatarMoveFlag: false
    }
  },
  computed: {
    ...mapState(['thirdParty', 'sodaAccount', 'audioPlayer', 'videoPlayer'])
  },
  watch: {
    // 监听路由跳转
    $route (to, from) {
      if (from.path === '/' && to.path === '/myMicLib/micMangage') {
        this.$refs.topNav.style.backgroundPositionX = parseFloat(this.getStyle(this.$refs.topNav, 'backgroundPositionX')) + 120 + 'px'
        this.$refs.topNavLib.style.color = '#0075c2'
      } else if (from.path === '/' && to.path === '/found') {
        this.$refs.topNavFound.style.color = '#0075c2'
      } else if (from.path === '/found' && to.path === '/myMicLib/micMangage') {
        // 改变导航条位置
        this.$refs.topNav.style.backgroundPositionX = parseFloat(this.getStyle(this.$refs.topNav, 'backgroundPositionX')) + 120 + 'px'
        this.$refs.topNavFound.style.color = 'rgba(0, 0, 0, 0.65)'
        this.$refs.topNavLib.style.color = '#0075c2'
      } else if (to.path === '/found') {
        this.$refs.topNav.style.backgroundPositionX = parseFloat(this.getStyle(this.$refs.topNav, 'backgroundPositionX')) - 120 + 'px'
        this.$refs.topNavLib.style.color = 'rgba(0, 0, 0, 0.65)'
        this.$refs.topNavFound.style.color = '#0075c2'
      }
    },

    'sodaAccount.loginFlag' (newValue, oldValue) {
      this.loginFlag = newValue
    },

    loginFlag (newValue, oldValue) {
      // 切换禁止滚动的类
      if (newValue) {
        document.body.classList.add('no-scroll')
      } else {
        document.body.classList.remove('no-scroll')
      }
    },

    'videoPlayer.isPlay' (newValue, oldValue) {
      // 切换禁止滚动的类
      if (newValue) {
        document.body.classList.add('no-scroll')
      } else {
        document.body.classList.remove('no-scroll')
      }
    }

  },
  methods: {
    ...mapMutations(['setUserInfo', 'changeLoginState', 'logout']),

    // CSS获取DOM样式不同浏览器兼容性适配
    getStyle (oElement, sName) {
      return oElement.currentStyle ? oElement.currentStyle[sName] : getComputedStyle(oElement, null)[sName]
    },

    // 退出登录
    async handleLogout () {
      // 请求登出
      const { data } = await this.$http.neteasecloudApi.login.logout()
      // 成功后 -> 执行本地退出操作
      if (data.code === 200) {
        this.logout('neteasecloud')
        this.$message.success('成功退出!')
      }

      // 失败捕获
      // ...
    },

    // 关闭窗口
    closeWindow () {
      // 关闭登录窗口
      if (this.loginFlag) this.loginFlag = false
      // 关闭注册窗口
      if (this.registerFlag) this.registerFlag = false
      // 关闭MV窗口
      if (this.videoPlayer.isStart) {
        this.$store.commit('changeVideoState', false)
      }
    },

    // 前往新窗口
    goNewWindow (value) {
      this.loginFlag = value === 'login'
      this.registerFlag = value === 'register'
    },

    // 头像点击 -> 点击第二个 -> 两个头像位置互换
    clickAvatar () {
      if (this.thirdParty.isLogin && this.sodaAccount.isLogin) {
        const frontIndex = this.avatarMoveFlag ? 1 : 0
        const backIndex = this.avatarMoveFlag ? 0 : 1
        const avatarList = document.querySelectorAll('.ant-avatar')
        // 前 -> 后
        const currentLeftBack = parseFloat(this.getStyle(avatarList[frontIndex], 'left'))
        avatarList[frontIndex].style.left = `${currentLeftBack + 15}px`
        avatarList[frontIndex].style.zIndex = 1
        // 后 -> 前
        const currentLeftFront = parseFloat(this.getStyle(avatarList[backIndex], 'left'))
        avatarList[backIndex].style.left = `${currentLeftFront - 15}px`
        avatarList[backIndex].style.zIndex = 2

        this.avatarMoveFlag = !this.avatarMoveFlag
      }
    }
  },
  created () {
    this.loginFlag = this.$store.state.sodaAccount.loginFlag
  },
  mounted () {}
}
</script>

<style lang='less' scoped>
.html-container{
  position: relative;
  width: 100vw;
  height: 100vh;
  background-color: #f7f9fc;
}

.top-banner{
  display: flex;
  width: 960px;
  justify-content: space-between;
  margin: 0 auto;
  padding: 0 20px;
  border-bottom: 1px solid #e8e8e8;

  li{
    height: 100px;
    line-height: 100px;
    font-size: 0;
    flex-shrink: 0;

    .web-logo{
      width: 170px;
      margin-right: 15px;
      vertical-align: middle;
      cursor: pointer;
    }
    .top-nav-container{
      display: inline-block;
      vertical-align: middle;
      height: 100px;
      background: linear-gradient(90deg, rgb(0, 117, 194) 5px , rgb(188, 232, 239)) no-repeat 15px bottom;
      background-size: 90px 4px;
      transition: background .5s;

      .top-nav{
        display: inline-block;
        width: 90px;
        margin: 0 15px;
        font-size: 22px;
        cursor: pointer;

        &:hover{
          color: @font-color-hover;
        }
      }
    }
  }
}

.nav-checked{
    border-bottom: 4px solid #0075c2;
    color: #0075c2;
}

.bannner-popover-ul{
  margin: 0;
  text-align: center;

  li{
      width: 115px;
      height: 45px;
      line-height: 45px;
      cursor: pointer;

      span{
        font-size: 15px;
        flex-shrink: 0;
      }
  }

  li:hover{
    background-color: #f2f5f6;
  }
}

.main-container{
  width: 100%;
}

/* 可以设置不同的进入和离开动画 */
/* 设置持续时间和动画函数 */
// #region
// .slide-fade-enter-active {
//   transition: all .4s cubic-bezier(0.65, 0, 0.35, 1);
// }
// .slide-fade-leave-active {
//   transition: all .2s cubic-bezier(1.0, 0.5, 0.8, 1.0);
// }
// .slide-fade-enter, .slide-fade-leave-to
// /* .slide-fade-leave-active for below version 2.1.8 */ {
//   transform: translateY(120px);
//   opacity: 0;
// }

.mask-fade-enter-active {
  transition: all .4s cubic-bezier(0.65, 0, 0.35, 1);
}
.mask-fade-leave-active {
  transition: all .5s;
}
.mask-fade-enter, .mask-fade-leave-to
/* .slide-fade-leave-active for below version 2.1.8 */ {
  opacity: 0;
}
// #endregion

.footer{
  background-color: #f2f2f2;

  .footer-content{
   display: flex;
   flex-direction: row;
   flex-wrap: wrap;
   justify-content: space-evenly;
   align-items: center;
   width: 1000px;
   height: 500px;
   margin: 0 auto;

    .footer-info{
      width: 140px;
      height: 300px;
    }

    .songs-link, .my-link{
      ul{
        position: relative;
        padding: 20px 0;

        .info-title{
          margin-bottom: 40px;
          font-size: 30px;
          font-weight: 500;
        }

        .info-link{
          display: block;
          height: 40px;
          line-height: 40px;
          margin-top: 10px;
          font-size: 18px;
          color: #999;
          cursor: pointer;
        }

        img{
          position: absolute;
          right: 0;
          width: 40px;
        }
      }
    }

    .other-link{

      ul{
        padding: 60px 0;

        .info-link{
          display: block;
          height: 40px;
          line-height: 40px;
          margin-top: 10px;
          font-size: 18px;
          color: #999;
          cursor: pointer;
        }
      }
    }
  }
}
</style>
<style lang="less" scoped>
.ant-avatar{
  position: relative;
  border: 1px solid @color-grey;
}

.avatar-container{
    display: inline-block;
    max-width: 55px;
    height: 40px;
    line-height: 40px;
    white-space: nowrap;
}
</style>
