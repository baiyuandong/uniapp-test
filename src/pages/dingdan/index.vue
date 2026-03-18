<template>
  <view class="min-h-screen bg-gray-100">
    <!-- 顶部 tabs -->
    <view class="bg-white px-4 pt-3">
      <view class="mb-3 flex items-center justify-between">
        <view class="flex text-sm space-x-6">
          <text
            v-for="(item, index) in tabs"
            :key="index"
            :class="tab === index ? 'text-orange-500 font-medium' : 'text-gray-500'"
            @click="tab = index"
          >
            {{ item }}
          </text>
        </view>
        <view class="flex items-center text-sm text-gray-500 space-x-3">
          <text>筛选</text>
          <text>开发票</text>
        </view>
      </view>

      <!-- 提示条 -->
      <view
        class="mb-3 flex items-center justify-between rounded-lg bg-orange-50 px-3 py-2"
      >
        <view class="flex items-center text-sm text-orange-500 space-x-2">
          <text>查看出行消费</text>
          <view class="h-1.5 w-1.5 rounded-full bg-orange-500" />
        </view>
        <text class="text-sm text-orange-500">去查看</text>
      </view>
    </view>

    <!-- 订单列表 -->
    <view class="px-4">
      <view
        v-for="item in orderList"
        :key="item.id"
        class="mb-4 rounded-xl bg-white p-4 shadow-sm"
      >
        <!-- 头部 -->
        <view class="mb-2 flex items-center justify-between">
          <view class="flex items-center space-x-2">
            <view
              class="h-6 w-6 flex items-center justify-center rounded bg-orange-500 text-xs text-white"
            >
              {{ item.typeIcon }}
            </view>
            <text class="text-sm text-gray-800 font-medium">
              {{ item.type }}
            </text>
          </view>
          <text class="text-xs text-gray-400">
            {{ item.status }}
          </text>
        </view>

        <!-- 行程信息 -->
        <view class="text-xs text-gray-500 space-y-1">
          <text>起点：{{ item.start }}</text>
          <text>终点：{{ item.end }}</text>
          <text>下单时间：{{ item.time }}</text>
        </view>

        <!-- 金额 -->
        <view
          v-if="item.price"
          class="mt-2 text-right text-base text-gray-800 font-medium"
        >
          ¥{{ item.price }}
        </view>

        <!-- 操作按钮 -->
        <view class="mt-3 flex justify-end space-x-2">
          <view
            v-for="btn in item.actions"
            :key="btn"
            class="rounded-full px-3 py-1.5 text-xs"
            :class="btn === '再来一单'
              ? 'bg-orange-500 text-white'
              : 'border border-gray-300 text-gray-500'"
          >
            {{ btn }}
          </view>
        </view>
      </view>
    </view>
  </view>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

const tab = ref(0)
const tabs = ['全部', '待出发', '待支付']

const orderList = ref([
  {
    id: 1,
    type: '惊喜特价',
    typeIcon: '惠',
    status: '已完成',
    start: '天通中苑1区东区-17号楼',
    end: '北京首都国际机场3号航站楼',
    time: '2026年01月24日 13:55',
    price: '62.60',
    actions: ['开发票', '一键退款', '再来一单'],
  },
  {
    id: 2,
    type: '打车',
    typeIcon: '车',
    status: '已关闭',
    start: '天通苑北社区卫生服务中心-东门',
    end: '东小口东方剑桥天通中苑幼儿园-东南门',
    time: '2026年01月20日 09:34',
    price: '',
    actions: ['再来一单'],
  },
  {
    id: 3,
    type: '滴滴轻享',
    typeIcon: '享',
    status: '已完成',
    start: '东方剑桥天通中苑幼儿园-东南门',
    end: '天通苑北社区卫生服务中心',
    time: '2026年01月20日 08:31',
    price: '16.60',
    actions: ['开发票', '再来一单'],
  },
])
</script>
