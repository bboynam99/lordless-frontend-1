<template>
  <div id="mobile-referral-referees" class="referral-referees-box">
    <transition name="ld-hide-fade" mode="out-in" @after-enter="afterEnter">
      <referral-referees-skeletion v-if="loading"/>
      <div v-else-if="referees.total" class="referral-referees-container">
        <h3 class="d-flex f-align-center referral-referees-title">
          <span class="inline-block">{{ referees.confirmed }} confirmed</span>
          <span class="inline-block">{{ referees.confirming }} confirming</span>
        </h3>
        <ul class="referral-referees-list">
          <li class="d-flex f-align-start referral-referees-item"
            v-for="(item, index) of referees.list" :key="index">
            <lordless-blockies :scale="5" :seed="item.referee.address"/>
            <div class="v-flex col-flex referees-item-info">
              <p class="text-break referees-item-address">{{ item.referee.address }}</p>
              <span class="inline-block referees-tx-status" :class="{ 'is-confirmed': item.refererTx.status === 1, 'is-confirming': item.refererTx.status === 0 }">
                <span class="inline-block">{{ item.refererTx.status === 1 ? 'Confirmed' : 'Confirming' }}</span>
              </span>
              <p class="referees-item-date">{{ item.refererTx.finishAt * 1000 | dateFormat('HH:mm MMM DD YYYY') }}</p>
            </div>
          </li>
        </ul>
        <div class="text-center referees-noMore-text">
          <p v-if="loadMoreLoading && !referees.noMore">
            <i class="el-icon-loading"></i>
          </p>
          <p v-else-if="referees.noMore">No more referees~</p>
        </div>
      </div>
      <div v-else class="text-center d-flex col-flex f-auto-center referral-referees-empty">
        <span class="relative inline-block line-height-0 referees-empty-icon">
          <svg>
            <use xlink:href="#icon-sitdown-human"/>
          </svg>
        </span>
        <h3>You have no referees.</h3>
        <p>Invite your friends to get material rewards and deposit boost.</p>
        <lordless-btn theme="blue-linear" class="referees-empty-btn">Invite friends</lordless-btn>
      </div>
    </transition>
  </div>
</template>

<script>
import ReferralRefereesSkeletion from '@/components/skeletion/_mobile/referral/referees'
import { getReferees } from 'api'

import { initLoadingMixins } from '@/mixins'
export default {
  name: 'referral-referees-component',
  mixins: [ initLoadingMixins ],
  data: () => {
    return {
      rendered: false,
      referees: {
        list: [],
        confirmed: 0,
        confirming: 0,
        total: 0,
        ps: 1,
        pn: 10,
        noMore: false
      },
      loadMoreLoading: false,
      scrollHandle: null
    }
  },
  components: {
    ReferralRefereesSkeletion
  },
  methods: {
    afterEnter () {
      this.scrollListenerFunc()
    },
    async initReferees () {
      this.loadMoreLoading = false
      this.loading = true
      const { list = [], pn = 1, ps = 10, confirmed = 0, confirming = 0, total = 0 } = (await this.getRefereeMethod({ pn: 1 })) || {}
      this.referees = {
        list,
        pn,
        ps,
        confirmed,
        confirming,
        total,
        noMore: total <= ps
      }
      this.loading = false
    },

    async getRefereeMethod () {
      try {
        const res = await getReferees()
        if (res.code === 1000 && res.data) {
          return res.data
        }
        return null
      } catch (err) {
        console.log('-------- err', err.message)
        return null
      }
    },
    // 获取更多 records
    async loadMoreReferees (cb) {
      this.loadMoreLoading = true
      const info = this.referees
      const pn = info.pn + 1
      const { list = [], ps = info.ps, confirmed = 0, confirming = 0, total = 0 } = (await this.getRefereeMethod({ pn })) || {}

      let noMore = false
      if (list.length < ps) {
        noMore = true
      }
      this.$set(this, 'referees', Object.assign({}, info, {
        list: info.list.concat(list),
        pn,
        ps,
        confirmed,
        confirming,
        total,
        noMore
      }))
      this.loadMoreLoading = false
      return cb && cb()
    },

    /**
     * scroll 监听事件
     */
    scrollListenerFunc ({ bool = false, bottom = 80, pHeight = document.body.offsetHeight } = {}) {
      this.scrollHandle && document.removeEventListener('scroll', this.scrollHandle)
      this.scrollHandle = null

      const box = document.getElementById('mobile-referral-referees')
      let bHeight = box ? box.offsetHeight : 0
      console.log(' --- scroll', bHeight, bottom, pHeight)
      // 如果 bHeight 不存在或者 bHeight - bottom 小于 pHeight, return
      if (!bHeight || bHeight - bottom < pHeight) return

      const handleFunc = () => {
        if (bool || this.planRecords.noMore) return
        if (!bHeight) bHeight = box.offsetHeight
        const scrollTop = document.documentElement.scrollTop || window.pageYOffset || document.body.scrollTop

        // 如果页面上滑到达指定条件位置
        // 读取更多数据
        if (scrollTop + pHeight + bottom > bHeight) {
          bool = true
          this.loadMoreRecords(() => {
            this.$nextTick(() => {
              bool = false
              bHeight = box.offsetHeight
            })
          })
        }
      }
      handleFunc()

      this.scrollHandle = handleFunc

      this.$nextTick(() => {
        document.addEventListener('scroll', this.scrollHandle)
      })
    }
  },
  async activated () {
    if (!this.rendered) {
      this.rendered = true
      return
    }
    await this.initReferees()
    this.$nextTick(() => this.scrollListenerFunc())
  },
  mounted () {
    this.$nextTick(() => this.initReferees())
  }
}
</script>

<style lang="scss" scoped>
  .referral-referees-box {
    padding-top: 44px;
    padding-bottom: 44px;
  }
  .referral-referees-container {
    margin-top: 16px;
  }
  .referral-referees-title {
    padding-left: 20px;
    font-size: 18px;
    color: #0B2A48;
    >span {
      &:not(:first-of-type) {
        position: relative;
        margin-left: 8px;
        padding-left: 9px;
        &::before {
          content: '';
          position: absolute;
          left: 0;
          top: 50%;
          width: 1px;
          height: 12px;
          background-color: #ddd;
          transform: translateY(-50%);
        }
      }
    }
  }
  .referral-referees-list {
    margin-top: 4px;
  }
  .referral-referees-item {
    padding-top: 20px;
    padding-left: 20px;
    background-color: #fff;
    &:not(:last-of-type) {
      .referees-item-info {
        border-bottom: 1px solid #ddd;
        box-sizing: border-box;
      }
    }
  }
  .referees-item-info {
    margin-left: 12px;
    padding-right: 20px;
    padding-bottom: 20px;
  }
  .referees-item-address {
    font-size: 16px;
    color: #555;
  }
  .referees-tx-status {
    margin-top: 8px;
    width: 61px;
    height: 22px;
    border-radius: 5px;
    color: #fff;
    overflow: hidden;
    >span {
      padding: 8px 12px;
      font-size: 20px;
      transform: scale(.5);
      transform-origin: 0 0;
    }
    &.is-confirmed {
      background-color: #0079FF;
    }
    &.is-confirming {
      background-color: #F5515F;
    }
  }
  .referees-item-date {
    margin-top: 6px;
    font-size: 14px;
    color: #bbb;
  }

  // referral-referees-empty
  .referral-referees-empty {
    padding: 30px;
    box-sizing: border-box;
    @include viewport-unit(min-height, 100vh, 94px);
    >h3 {
      margin-top: 32px;
      font-size: 20px;
      color: #0B2A48;
    }
    >p {
      margin-top: 12px;
      font-size: 16px;
      color: #555;
    }
  }
  .referees-empty-icon {
    width: 298px;
    max-width: 100%;
    padding-top: 298px;
    >svg {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
  }
  .referees-empty-btn {
    margin-top: 32px;
    padding: 12px 20px;
  }

  .referees-noMore-text {
    padding-bottom: 16px;
    padding-top: 16px;
    font-size: 16px;
    color: #999;
    background-color: #fff;
    &::before {
      content: '';
      position: absolute;
      top: 0;
      left: 20%;
      width: 60%;
      height: 1px;
      background-color: #eee;
    }
  }
</style>