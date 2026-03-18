<template>
  <view class="h-screen bg-gray-100">
    <!-- Tabs -->
    <wd-tabs v-model="tab">
      <wd-tab
        v-for="(item, index) in tabs"
        :key="index"
        :title="item"
      >
        <!-- 列表 -->
        <scroll-view
          scroll-y
          class="h-[calc(100vh-96rpx)] bg-#f6f8fa px-4 py-3"
          @scrolltolower="loadMore"
        >
          <!-- 优惠券 -->
          <view
            v-for="coupon in couponList"
            :key="coupon.id"
            class="relative mb-4 flex overflow-hidden rounded-xl bg-white"
          >
            <!-- 左侧折扣区（白底） -->
            <view
              class="w-24 flex flex-col items-center justify-center text-orange-500"
            >
              <view class="flex items-end leading-none">
                <text class="text-3xl font-bold">
                  {{ coupon.discount }}
                </text>
                <text class="mb-0.5 ml-0.5 text-sm">折</text>
              </view>
            </view>

            <!-- 中间虚线 + 撕口 -->
            <view class="relative flex items-center">
              <!-- 上半圆 -->
              <view
                class="absolute left-1/2 z-10 h-4 w-4 rounded-full bg-gray-100 -top-2 -translate-x-1/2"
              />

              <!-- 虚线 -->
              <view class="h-full border-l border-gray-300 border-dashed" />

              <!-- 下半圆 -->
              <view
                class="absolute left-1/2 z-10 h-4 w-4 rounded-full bg-gray-100 -bottom-2 -translate-x-1/2"
              />
            </view>

            <!-- 右侧内容区 -->
            <view class="flex-1 px-4 py-3">
              <!-- 标题 + 按钮 -->
              <view class="flex items-start justify-between">
                <view class="pr-2">
                  <view class="text-sm text-gray-800 font-medium">
                    {{ coupon.title }}
                  </view>
                  <view class="mt-1 text-xs text-gray-400">
                    {{ coupon.desc }}
                  </view>
                </view>

                <wd-button
                  size="small"
                  type="primary"
                  :disabled="tab !== 0"
                  class="!px-3 !py-1"
                >
                  立即使用
                </wd-button>
              </view>

              <!-- 倒计时 -->
              <view class="mt-2 text-xs text-orange-500">
                仅剩 {{ coupon.remainTime }}
              </view>
            </view>
          </view>

          <!-- 没有更多 -->
          <view
            v-if="finished"
            class="py-6 text-center text-xs text-gray-400"
          >
            没有更多优惠券了
          </view>
        </scroll-view>
      </wd-tab>
    </wd-tabs>
  </view>
</template>

<script lang="ts" setup>
import { reactive, ref, watch } from 'vue'

const tab = ref(0)
const tabs = ['未使用', '已使用', '已过期']

const loadingMore = ref(false)
const finished = ref(false)

const queryParams = reactive({
  pageNo: 1,
  pageSize: 20,
})

const couponList = ref<any[]>([])

definePage({
  style: {
    navigationBarTitleText: '优惠券',
  },
})

// 模拟接口
function loadCoupons() {
  loadingMore.value = true

  setTimeout(() => {
    const list = [
      {
        id: Date.now(),
        discount: '7.5',
        title: '专车机场/火车站折扣券',
        desc: '仅机场/火车站专车订单可用',
        remainTime: '08:28:03',
      },
      {
        id: Date.now() + 1,
        discount: '9',
        title: '滴滴轻享折扣券',
        desc: '限滴滴轻享可用，最高可减20元',
        remainTime: '08:28:03',
      },
      {
        id: Date.now() + 2,
        discount: '8.7',
        title: '轻享通勤折扣券',
        desc: '限滴滴轻享可用，最高可减20元',
        remainTime: '04:27:04',
      },
    ]

    couponList.value.push(...list)
    loadingMore.value = false
    finished.value = true
  }, 500)
}

// 上拉加载
function loadMore() {
  if (loadingMore.value || finished.value)
    return
  queryParams.pageNo++
  loadCoupons()
}

// tab 切换
watch(tab, () => {
  couponList.value = []
  queryParams.pageNo = 1
  finished.value = false
  loadCoupons()
})

// 初始加载
loadCoupons()
</script>
