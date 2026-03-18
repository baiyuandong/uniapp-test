<template>
  <view class="min-h-screen bg-gray-100">
    <!-- 顶部 -->
    <view class="bg-gradient-to-r from-blue-500 to-blue-600 px-4 py-3 flex items-center justify-between">
      <text class="text-white text-lg font-semibold">电子发票</text>
      <view class="text-white text-sm px-3 py-1 bg-white/20 rounded-full">
        抬头管理
      </view>
    </view>

    <!-- Tab -->
    <view class="bg-white flex">
      <view class="flex-1 text-center py-3 font-medium border-b-2 border-blue-500 text-blue-500">
        待开票
      </view>
      <view class="flex-1 text-center py-3 text-gray-400">
        已开票
      </view>
    </view>

    <!-- 订单卡片 -->
    <view class="p-4">
      <view class="bg-white rounded-xl p-4 shadow-sm space-y-2">
        <view class="flex justify-between text-sm text-gray-500">
          <text>产品订单</text>
          <text>2025-12-01 13:46:50</text>
        </view>

        <view class="flex justify-between items-center">
          <text class="font-semibold text-gray-800">西安·敦煌7日游（研学）</text>
          <text class="text-xs px-2 py-1 rounded-full bg-orange-100 text-orange-500">
            待开票
          </text>
        </view>

        <text class="text-sm text-gray-600">
          出行人数：1成人，1儿童
        </text>

        <view class="flex justify-between items-center pt-2">
          <text class="text-red-500 font-semibold">
            ¥900.00
          </text>
          <button class="bg-blue-500 text-white text-sm px-5 py-2 rounded-full">
            去开具
          </button>
        </view>
      </view>
    </view>
  </view>
</template>
