<template>
  <el-container class="ld-user-box">
    <aside class="ld-user-navgation">
    <!-- <aside class="d-flex lg-col-flex sm-row-flex ld-user-navgation"> -->
      <div class="d-flex f-auto-center user-navgation-logo sm-hidden">
        <lordless-header-logo theme="light" shadow reverse/>
      </div>
      <div class="overflow user-navgation-cnt">
        <div class="relative user-navgation-container" :style="`background-image: url(${ossOrigin}/frontend/static/svg/city.svg)`">
          <ul class="d-flex sm-row-flex lg-col-flex user-navgation-list">
            <li
              class="user-navgation-item"
              :class="{ 'active': $route.path.match(nav.match) }"
              v-for="(nav, index) of navgations"
              :key="index"
              @click="$router.push(nav.path)">
              <div class="d-flex f-align-center sm-f-justify-center TTFontBolder navgation-item-cnt">
                <span class="navgation-item-icon">
                  <svg>
                    <use :xlink:href="$route.meta.navgation === nav.name.toLowerCase() ? nav.selectIcon : nav.unSelectIcon"/>
                  </svg>
                </span>
                <span class="text-cap">{{ nav.name }}</span>
              </div>
            </li>
          </ul>
          <div class="TTFontBolder user-navgation-item user-logout sm-hidden">
            <div class="d-flex f-align-center cursor-pointer navgation-item-cnt">
              <span class="navgation-item-icon">
                <svg>
                  <use xlink:href="#icon-logout"/>
                </svg>
              </span>
              <span class="text-cap" @click="logout">Logout</span>
            </div>
          </div>
        </div>
      </div>
    </aside>
    <el-container class="col-flex ld-user-container">
      <div class="ld-user-header">
        <ld-header v-bind="headerOpt" theme="light"></ld-header>
      </div>
      <el-main id="user-main-content" class="d-flex ld-user-content">
        <div class="v-flex d-flex user-content-container">
          <transition name="owner-cnt-fade">
            <router-view class="owner-cnt-box" v-if="true"></router-view>
          </transition>
        </div>
      </el-main>
    </el-container>
    <lordless-authorize
      ref="ownerAuthorize"
      :modelClose="false"
      :autoClose="false"
      blurs
      @init="checkUser"
      @fClose="fCloseAuthorize"/>
  </el-container>
</template>

<script>
import LdHeader from '@/components/content/layout/header'
// import Authorize from '@/components/reuse/dialog/authorize'
// import Loading from '@/components/stories/loading'

import { userMixins, publicMixins } from '@/mixins'
export default {
  mixins: [ userMixins, publicMixins ],
  data: () => {
    return {
      navgations: [
        {
          name: 'overview',
          selectIcon: '#icon-overview_selected',
          unSelectIcon: '#icon-overview_unselected',
          path: '/owner/info',
          match: /^\/owner\/info/
        },
        // {
        //   name: 'candy',
        //   selectIcon: '#icon-candy_selected',
        //   unSelectIcon: '#icon-candy_unselected',
        //   path: '/owner/candy',
        //   match: /^\/owner\/candy/
        // },
        {
          name: 'HOPS',
          selectIcon: '#coin-hops',
          unSelectIcon: '#coin-hops',
          path: '/owner/hops',
          match: /^\/owner\/hops/
        },
        {
          name: 'Bounty',
          selectIcon: '#icon-bounty-chests',
          unSelectIcon: '#icon-bounty-chests',
          path: '/owner/bounty/package',
          match: /^\/owner\/bounty\/package/
        },
        {
          name: 'BC',
          selectIcon: '#icon-tab-bc',
          unSelectIcon: '#icon-tab-bc',
          path: '/owner/bc',
          match: /^\/owner\/bc/
        },
        {
          name: 'Invitation',
          selectIcon: '#icon-tab-diploma',
          unSelectIcon: '#icon-tab-diploma',
          path: '/owner/invitation',
          match: /^\/owner\/invitation/
        },
        {
          name: 'Referee',
          selectIcon: '#icon-badge',
          unSelectIcon: '#icon-badge',
          path: '/owner/referee',
          match: /^\/owner\/referee/
        },
        {
          name: 'quests',
          selectIcon: '#icon-quests_selected',
          unSelectIcon: '#icon-quests_unselected',
          path: '/owner/quests',
          match: /^\/owner\/quests/
        },
        {
          name: 'taverns',
          selectIcon: '#icon-taverns_selected',
          unSelectIcon: '#icon-taverns_unselected',
          path: '/owner/taverns',
          match: /^\/owner\/taverns/
        },
        // {
        //   name: 'activity',
        //   icon: 'activity',
        //   path: '/owner/activity'
        // },
        {
          name: 'authorization',
          selectIcon: '#icon-authorization_selected',
          unSelectIcon: '#icon-authorization_unselected',
          path: '/owner/authorization',
          match: /^\/owner\/authorization/
        }
      ],
      headerOpt: {
        show: true,
        logo: {
          show: true,
          mobile: true,
          pc: false
        },
        showLogo: false,
        inverse: false,
        transparent: true,
        inherit: true,
        zIndex: 99
      },
      loading: true
    }
  },
  computed: {
    ossOrigin () {
      return process.env.LDBICON_ORIGIN
    }
  },
  components: {
    // Loading,

    LdHeader
    // Authorize
  },
  methods: {

    fCloseAuthorize () {
      if (this.loading) this.$router.push('/')
    },

    async checkUser () {
      const authorize = await this.$refs.ownerAuthorize.checkoutAuthorize()
      this.loading = !this.userInfo.address && !authorize
    }
  },
  watch: {
    account (val, oVal) {
      console.log('---- owner info account watch', val, oVal)
      this.$nextTick(() => this.checkUser())
    }
  },
  mounted () {
    this.$nextTick(() => this.checkUser())
  }
}
</script>

<style lang="scss">
  @import '@/assets/stylus/reuse/single/user_info.scss';
</style>

<style lang="scss" scoped>
  .owner-cnt-fade-enter-active {
    opacity: 1;
    z-index: 1;
    transition: opacity .4s cubic-bezier(0.4, 0, 0.2, 1);
  }
  .owner-cnt-fade-leave-active {
    transition-duration: 0s;
  }
  .owner-cnt-fade-enter, .owner-cnt-fade-leave-to {
    opacity: 0;
  }

  .ld-user-box {
    height: 100vh;
    min-height: 500px;
  }

  // ld-user-navgation
  .ld-user-navgation {
    width: 250px;
    min-height: 670px;
    z-index: 1;
    box-shadow: 1px 3px 5px 0 rgba(0, 0, 0, .1);
    // @include viewport-unit(min-height, 100vh, 90px);
  }

  .user-navgation-logo {
    height: 90px;
    // border-bottom: 1px solid rgba(77, 70, 210, 0.05);
    fill: $--text-deep-blue-color;
    cursor: pointer;
    svg {
      width: 150px;
    }
  }

  .user-navgation-cnt {
    @include viewport-unit(height, 100vh, 80px);
  }
  .user-navgation-container {
    padding-bottom: 260px;
    background-size: 100% auto;
    background-position-y: 100%;
    background-repeat: no-repeat;
    background-color: #fff;
    box-sizing: broder-box;
    @include viewport-unit(min-height, 100vh, 80px);
  }
  .user-navgation-item {
    position: relative;
    font-size: 16px;
    color: #999;
    fill: #999;
    span {
      display: inline-block;
    }
    &:not(.user-logout) {
      cursor: pointer;
    }
    &.active {
      color: $--text-blue-color;
      fill: $--text-blue-color;
      background-color: #f8f8f8;
      .navgation-item-cnt {
        &::before {
          visibility: visible;
        }
      }
    }
    &:hover {
      &:not(.user-logout):not(.active) {
        background-color: rgba(77, 70, 210, 0.02);
      }
    }
  }
  .navgation-item-cnt {
    position: relative;
    width: 100%;
    box-sizing: border-box;
    @include padding('left', 20px, -2);
  }
  .navgation-item-icon {
    width: 22px;
    height: 22px;
    @include margin('right', 15px, -2);
  }
  .user-logout {
    position: absolute;
    left: 0;
    bottom: 15%;
    width: 100%;
    height: 100px;
    // color: $--text-red-color;
    // fill: $--text-red-color;
    color: #F5515F;
    fill: #F5515F;
  }

  .ld-user-container {
    position: relative;
    z-index: 0;
    @include padding('bottom', 60px, 1, -2);
  }
  .ld-user-header {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    padding: 0 30px;
    background-color: #fff;
    // color: #fff;
    z-index: 9;
    box-shadow: 2px 3px 5px 0 rgba(0, 0, 0, .1);
    @include height(90px, -2);
    @include height(70px, 1, -2);
  }

  .ld-user-content {
    padding: 0;
    @include padding('top', 90px, -2);
    @include padding('top', 70px, 1, -2);
    background-color: #f4f4f4;
    @include overflow();
  }
  .user-content-container {
    position: relative;
    margin: 0 auto;
    max-width: 1000px;
    @include viewport-unit(width, 100vw);
  }
  .owner-cnt-box {
    padding: 30px;
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
    min-height: 100%;
    box-sizing: border-box;
    // @include overflow();
  }
  @media screen and (max-width: 991px) {
    .ld-user-navgation {
      width: 100px;
    }
    .user-navgation-logo {
      // .logo-text {
      //   display: none;
      // }
      svg {
        width: 50px;
      }
    }
    .text-cap {
      display: none !important;
    }
  }
  @media screen and (min-width: 769px) {
    .user-navgation-item {
      padding: 20px 0;
    }
    .navgation-item-cnt {
      &::before {
        content: '';
        position: absolute;
        left: 0;
        top: 0;
        width: 5px;
        height: 100%;
        background-color: $--text-blue-color;
        visibility: hidden;
      }
    }
  }
  @media screen and (max-width: 768px) {
    .ld-user-navgation {
      position: absolute;
      width: 100%;
      height: 60px;
      bottom: 0;
      left: 0;
      z-index: 99;
      background-color: #fff;
    }
    .user-navgation-item {
      text-align: center;
      flex: 1;
    }
    .navgation-item-cnt {
      height: 100%;
    }
  }
</style>
