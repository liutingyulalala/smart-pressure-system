<template>
  <el-row :gutter="40" class="panel-group">
    <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
      <div class="card-panel">
        <div class="card-panel-icon-wrapper icon-people">
          <i v-if="reasoningResult.diff_value > 0" class="iconfont icon-a-gaoxinghaoxinqingbiaoqingxihuanbiaoqingxiaolian card-panel-icon" />
          <i v-else class="iconfont icon-a-dakubeishangshangxinbiaoqingxiaolian card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            {{ isThroughput ? '--' : reasoningResult.diff_value > 0 ? '增益' : '耗损' }}
          </div>
          <count-to :start-val="0" :end-val="reasoningResult.diff_value || 0" :duration="3600" class="card-panel-num" />
          <span>元 / 年</span>
        </div>
      </div>
    </el-col>
    <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
      <div class="card-panel">
        <div class="card-panel-icon-wrapper icon-money">
          <svg-icon icon-class="money" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            单价
          </div>
          <div class="card-panel-counts">
            <div class="card-panel-counts__items">
              <count-to :start-val="0" :end-val="reasoningResult.unit_price.cpu || 0" :duration="3200" class="card-panel-num" />
              <span class="card-panel-counts__label">/ C</span>
            </div>
            <div class="card-panel-counts__items">
              <count-to :start-val="0" :end-val="reasoningResult.unit_price.mem || 0" :duration="3200" class="card-panel-num" />
              <span class="card-panel-counts__label">/ G</span>
            </div>
          </div>
        </div>
      </div>
    </el-col>
    <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
      <div class="card-panel">
        <div class="card-panel-icon-wrapper icon-shopping" style="padding: 8px 14px;">
          <i class="iconfont icon-yu card-panel-icon" style="font-size: 55px;" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            预测值
          </div>
          <div class="card-panel-counts">
            <div class="card-panel-counts__items">
              <span class="card-panel-num">{{ reasoningResult.pred_value.pod_cores || 0 }}</span>
              <span class="card-panel-counts__label"> C</span>
            </div>
            <div class="card-panel-counts__items">
              <span class="card-panel-num">{{ reasoningResult.pred_value.pod_memory || 0 }}</span>
              <span class="card-panel-counts__label"> G</span>
            </div>
          </div>
        </div>
      </div>
    </el-col>
    <el-col :xs="12" :sm="12" :lg="6" class="card-panel-col">
      <div class="card-panel">
        <div class="card-panel-icon-wrapper icon-message">
          <svg-icon icon-class="dashboard" class-name="card-panel-icon" />
        </div>
        <div class="card-panel-description">
          <div class="card-panel-text">
            当前值
          </div>
          <div class="card-panel-counts">
            <div class="card-panel-counts__items">
              <span class="card-panel-num">{{ reasoningResult.pre_alloc.pod_cores || 0 }}</span>
              <span class="card-panel-counts__label"> C</span>
            </div>
            <div class="card-panel-counts__items">
              <span class="card-panel-num">{{ reasoningResult.pre_alloc.pod_memory || 0 }}</span>
              <span class="card-panel-counts__label"> G</span>
            </div>
          </div>
        </div>
      </div>
    </el-col>
  </el-row>
</template>

<script>
import CountTo from 'vue-count-to'
import { mapState } from 'vuex'

export default {
  components: {
    CountTo
  },
  computed: {
    ...mapState({
      reasoningResult: state => state.app.reasoningResult
    }),
    isThroughput() {
      return this.reasoningResult.modelCate === 'throughput'
    }
  },
  methods: {
    handleSetLineChartData(type) {
      this.$emit('handleSetLineChartData', type)
    }
  }
}
</script>

<style lang="scss" scoped>
.card-panel-counts {
  &__items {
    margin-bottom: 5px;
  }
  &__label {
    color: #d95151;
  }
  .card-panel-num {
    font-size: 14px !important;
  }

}
.panel-group {
  margin-top: 18px;

  .card-panel-col {
    margin-bottom: 32px;
  }

  .card-panel {
    height: 108px;
    cursor: pointer;
    font-size: 12px;
    position: relative;
    overflow: hidden;
    color: #666;
    background: #fff;
    box-shadow: 4px 4px 40px rgba(0, 0, 0, .05);
    border-color: rgba(0, 0, 0, .05);

    &:hover {
      .card-panel-icon-wrapper {
        color: #fff;
      }

      .icon-people {
        background: #ffba00;
      }

      .icon-message {
        background: #36a3f7;
      }

      .icon-money {
        background: #f4516c;
      }

      .icon-shopping {
        background: #5b7ee7;
      }
    }

    .icon-people {
      color: #ffba00;
    }

    .icon-message {
      color: #36a3f7;
    }

    .icon-money {
      color: #f4516c;
    }

    .icon-shopping {
      color: #5b7ee7;
    }

    .card-panel-icon-wrapper {
      float: left;
      margin: 14px 0 0 14px;
      padding: 16px;
      transition: all 0.38s ease-out;
      border-radius: 6px;
    }

    .card-panel-icon {
      float: left;
      font-size: 48px;
    }

    .card-panel-description {
      float: right;
      font-weight: bold;
      margin: 22px 30px;
      margin-left: 0px;

      .card-panel-text {
        line-height: 18px;
        color: rgba(0, 0, 0, 0.45);
        font-size: 16px;
        margin-bottom: 12px;
      }

      .card-panel-num {
        font-size: 20px;
      }
    }
    .icon-money .card-panel-description .card-panel-text {
      margin-bottom: 5px;
    }
  }
}

@media (max-width:550px) {
  .card-panel-description {
    display: none;
  }

  .card-panel-icon-wrapper {
    float: none !important;
    width: 100%;
    height: 100%;
    margin: 0 !important;

    .svg-icon {
      display: block;
      margin: 14px auto !important;
      float: none !important;
    }
  }
}
</style>
