<template>
  <div id="mobile-plan-records" class="mobile-plan-records">
    <transition name="ld-hide-fade" mode="out-in" @after-enter="afterEnter">
      <plan-records-skeletion v-if="loading"/>
      <div v-else-if="!loading && !planRecords.total" class="d-flex f-auto-center text-center mobile-deposits-none">
        <div class="deposits-none-container">
          <span class="relative inline-block line-height-0 deposits-none-icon">
            <svg>
              <use xlink:href="#icon-human2"/>
            </svg>
          </span>
          <h3>You have no HOPS records.</h3>
          <p>Deposit your LESS and reap HOPS immediately.</p>
          <lordless-btn theme="blue-linear" class="deposits-none-btn" @click="$router.push('/owner/hops')">Deposit LESS</lordless-btn>
        </div>
      </div>
      <div v-else class="plan-records-container">
        <ul>
          <li v-for="(record, index) of planRecords.list" :key="index" class="plan-records-item" :class="{ 'none-line': record.month }">
            <p v-if="record.month" class="plan-records-date">{{ record.month.timestamp | dateFormat('MMM YYYY') }}</p>
            <plan-record-card :user="userAddress" :info="record" :recordType="record.type"/>
          </li>
        </ul>
        <div class="text-center records-noMore-text">
          <p v-if="loadMoreLoading && !planRecords.noMore">
            <i class="el-icon-loading"></i>
          </p>
          <p v-else-if="planRecords.noMore">No more planRecords~</p>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
import PlanRecordsSkeletion from '@/components/skeletion/_mobile/hops/records'
import PlanRecordCard from '@/components/reuse/_mobile/card/plan/record'

import { getLastYearMonths } from 'utils/tool'

import { getUserPlanRecords } from 'api'
import { initLoadingMixins, publicMixins } from '@/mixins'
export default {
  name: 'mobile-plan-records-content',
  mixins: [ initLoadingMixins, publicMixins ],
  data: () => {
    return {
      rendered: false,
      loadMoreLoading: false,
      planRecords: {
        list: [],
        pn: 1,
        ps: 10,
        total: 0,
        noMore: false
      }
    }
  },
  watch: {
    'planRecords.list' (val) {
      this.rewriteRecords(val)
      console.log('------ watch planRecords.list')
    }
  },
  computed: {
    userAddress () {
      return this.$route.params.address || this.account
    }
  },
  components: {
    PlanRecordsSkeletion,
    PlanRecordCard
  },
  methods: {
    afterEnter () {
      this.scrollListenerFunc()
    },

    rewriteRecords (list = this.planRecords.list) {
      const lastYearMonths = getLastYearMonths()
      for (let item of list) {
        if (item.month) item.month = null
        if (!lastYearMonths.length) continue
        for (let i = 0; i < lastYearMonths.length; i++) {
          if (item.recordsAt * 1000 < lastYearMonths[i].nextTimestamp) {
            item.month = lastYearMonths[i]
            lastYearMonths.splice(i, 1)
            break
          }
        }
      }
    },

    async initPlanRecords () {
      this.loadMoreLoading = false
      this.loading = true
      const { list = [], pn = 1, ps = 10, total = 0 } = (await this.getUserPlanRecords({ pn: 1 })) || {}
      this.planRecords = {
        list,
        pn,
        ps,
        total,
        noMore: total <= ps
      }
      this.loading = false
      if (!this.rendered) this.rendered = true
    },

    async getUserPlanRecords ({ pn, ps = this.planRecords.ps } = {}) {
      try {
        const res = await getUserPlanRecords({ pn, ps, type: 'all' })
        if (res.code === 1000 && res.data) {
          return res.data
        }
        return null
      } catch (err) {
        console.log('---- err', err)
        return null
      }
    },

    // 获取更多 records
    async loadMoreRecords (cb) {
      this.loadMoreLoading = true
      const info = this.planRecords
      const pn = info.pn + 1
      const { list = [], ps = info.ps, total = 0 } = (await this.getUserPlanRecords({ pn })) || {}

      let noMore = false
      if (list.length < ps) {
        noMore = true
      }
      this.$set(this, 'planRecords', Object.assign({}, info, {
        list: info.list.concat(list),
        pn,
        ps,
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

      const box = document.getElementById('mobile-plan-records')
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
    if (!this.rendered) return
    await this.initPlanRecords()

    this.$nextTick(() => this.scrollListenerFunc())
  },
  mounted () {
    this.$nextTick(() => this.initPlanRecords())
  }
}
</script>

<style lang="scss" scoped>
  /**
   *  mobile-deposits-none  -- begin
   */
  .mobile-deposits-none {
    padding: 30px;
    box-sizing: border-box;
    @include viewport-unit(min-height, 100vh, 94px);
  }
  .deposits-none-container {
    margin: 0 auto;
    max-width: 300px;
    >h3 {
      margin-top: 30px;
      font-size: 20px;
    }
    >p {
      margin-top: 12px;
      font-size: 16px;
      color: #555;
    }
  }
  .deposits-none-icon {
    width: 100%;
    padding-top: 100%;
    >svg {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
    }
  }
  .deposits-none-btn {
    margin-top: 32px;
    padding: 12px 20px;
  }
  /**
   *  mobile-deposits-none  -- end
   */
  .mobile-plan-records {
    margin-top: 44px;

  }
  .plan-records-container {

  }
  .plan-records-item {
    &:not(:first-of-type) {
      &:not(.none-line) {
        /deep/ {
        .plan-record-info {
            border-top: 1px solid #ddd;
          }
        }
      }
    }
  }
  .plan-records-date {
    padding-top: 12px;
    padding-bottom: 4px;
    padding-left: 24px;
    font-size: 16px;
  }
  .records-noMore-text {
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
