<template>
  <view class="box-border flex h-screen flex-col bg-#f5f6f8">
    <!-- 顶部：状态栏占位 + 当前节点 / 智能代理 -->
    <view class="shrink-0 bg-white pb-2 pl-3 pr-3 pt-safe">
      <view class="flex items-center justify-between pt-2">
        <view class="flex max-w-[50%] items-center gap-2">
          <view class="relative h-10 w-10 flex shrink-0 items-center justify-center overflow-hidden rounded-lg bg-#f0f2f5 text-xl">
            {{ currentNode.flag }}
            <view
              v-if="currentNode.connectedMark"
              class="absolute left-0 top-0 h-4 w-4 flex items-center justify-center rounded-br bg-#22c55e text-10px text-white leading-none"
            >
              ✓
            </view>
          </view>
          <view class="min-w-0">
            <view class="truncate text-15px text-#111 font-semibold">
              {{ currentNode.name }}
            </view>
            <view class="mt-0.5 flex items-center gap-1 text-11px text-#22c55e">
              <text>▮▮▮▯</text>
            </view>
          </view>
        </view>
        <view class="flex shrink-0 items-center gap-2">
          <view class="text-right">
            <view class="text-14px text-#111 font-medium">
              智能代理
            </view>
            <view class="text-11px text-#22c55e">
              已连接
            </view>
          </view>
          <text class="px-1 text-20px text-#666 leading-none">⚙</text>
        </view>
      </view>

      <!-- 活动横幅 -->
      <scroll-view scroll-x class="mt-2 w-full" :show-scrollbar="false">
        <view class="inline-flex py-1">
          <text class="whitespace-nowrap text-12px text-orange-500 leading-relaxed">
            活动 & 涨价通知 ❤️：各位五一假期快乐！感谢一直以来的支持～
          </text>
        </view>
      </scroll-view>
    </view>

    <!-- 列表区 -->
    <view class="mt-2 shrink-0 px-3">
      <view class="flex items-center justify-between py-2">
        <view class="flex items-center gap-1 text-14px text-#333 font-semibold">
          <text>IEPL专线</text>
          <text class="text-12px text-#999">ⓘ</text>
        </view>
        <text class="text-12px text-#999">
          了解更多 &gt;
        </text>
      </view>
    </view>

    <scroll-view scroll-y class="min-h-0 flex-1" :show-scrollbar="false">
      <view class="box-border px-3 pb-220rpx">
        <view
          v-for="node in nodeList"
          :key="node.id"
          class="relative mb-3 overflow-hidden rounded-2xl bg-white p-3 shadow-sm"
          :class="node.selected ? 'bg-#fffbeb ring-1 ring-amber-200' : ''"
          @click="selectNode(node)"
        >
          <view class="flex gap-3">
            <view class="relative h-11 w-11 flex shrink-0 items-center justify-center overflow-hidden rounded-lg bg-#f0f2f5 text-22px">
              {{ node.flag }}
              <view
                v-if="node.activeMark"
                class="absolute left-0 top-0 h-4 w-4 flex items-center justify-center rounded-br bg-#22c55e text-10px text-white leading-none"
              >
                ✓
              </view>
            </view>

            <view class="min-w-0 flex-1">
              <view class="flex flex-wrap items-center gap-1.5">
                <text class="text-15px text-#111 font-semibold">{{ node.name }}</text>
                <text
                  v-for="(tag, ti) in node.tags"
                  :key="ti"
                  class="rounded px-1.5 py-0.5 text-10px leading-none"
                  :class="tagClass(tag.tone)"
                >
                  {{ tag.text }}
                </text>
              </view>
              <view class="mt-2 flex flex-wrap gap-x-3 gap-y-1 text-11px text-#888">
                <text>带宽 {{ node.bandwidth }}</text>
                <text>倍率 {{ node.multiplier }}</text>
                <text>延迟 {{ node.latency }}</text>
                <text :class="loadClass(node.loadTone)">
                  负载 {{ node.load }}
                </text>
              </view>
            </view>

            <view
              class="flex shrink-0 items-start pt-0.5 text-20px text-#ccc"
              @click.stop="toggleStar(node)"
            >
              {{ node.starred ? '★' : '☆' }}
            </view>
          </view>
        </view>
      </view>
    </scroll-view>

    <!-- 底部导航 + 中间连接球 -->
    <view class="shrink-0 border-t border-#eee bg-white pb-safe">
      <view class="relative h-112rpx">
        <view class="flex h-full items-end justify-around pb-12rpx pt-8rpx">
          <view class="flex flex-col items-center text-11px text-amber-500">
            <text class="mb-1 text-20px">
              ◈
            </text>
            <text>节点</text>
          </view>
          <view class="w-20" />
          <view class="flex flex-col items-center text-11px text-#999">
            <text class="mb-1 text-20px">
              ◉
            </text>
            <text>我的</text>
          </view>
        </view>
        <view
          class="absolute bottom-40rpx left-1/2 z-10 h-120rpx w-120rpx flex -translate-x-1/2 flex-col items-center justify-center rounded-full bg-#22c55e text-center text-white shadow-lg"
        >
          <text class="text-13px font-medium">已连接</text>
          <text class="mt-1 text-11px opacity-90">剩余128G</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script lang="ts" setup>
import { ref } from 'vue'

defineOptions({
  name: 'ProxyNodesDemo',
})

type TagTone = 'pink' | 'green' | 'purple' | 'gray' | 'blue' | 'orange'

interface NodeTag {
  text: string
  tone: TagTone
}

type LoadTone = 'green' | 'orange' | 'default'

interface ProxyNode {
  id: string
  flag: string
  name: string
  tags: NodeTag[]
  bandwidth: string
  multiplier: string
  latency: string
  load: string
  loadTone: LoadTone
  starred: boolean
  selected?: boolean
  activeMark?: boolean
}

definePage({
  style: {
    navigationStyle: 'custom',
    navigationBarTitleText: '节点',
  },
})

const currentNode = ref({
  flag: '🇯🇵',
  name: '日本 I0',
  connectedMark: true,
})

const nodeList = ref<ProxyNode[]>([
  {
    id: '1',
    flag: '🇭🇰',
    name: '香港 10D',
    tags: [
      { text: 'AWS', tone: 'pink' },
      { text: 'IPv6', tone: 'green' },
    ],
    bandwidth: '1G',
    multiplier: '2x',
    latency: '-',
    load: '49.9M/s',
    loadTone: 'green',
    starred: false,
  },
  {
    id: '2',
    flag: '🇭🇰',
    name: '香港 I1',
    tags: [
      { text: 'IEPL', tone: 'purple' },
      { text: '豪华专享', tone: 'gray' },
    ],
    bandwidth: '1G',
    multiplier: '3x',
    latency: '-',
    load: '25.9M/s',
    loadTone: 'green',
    starred: false,
  },
  {
    id: '3',
    flag: '🇭🇰',
    name: '香港 I2',
    tags: [
      { text: 'AWS', tone: 'pink' },
      { text: 'IPv6', tone: 'green' },
      { text: '动态IP', tone: 'blue' },
      { text: '新开机', tone: 'blue' },
    ],
    bandwidth: '1G',
    multiplier: '2x',
    latency: '-',
    load: '83.8M/s',
    loadTone: 'orange',
    starred: false,
  },
  {
    id: '4',
    flag: '🇭🇰',
    name: '香港 I4',
    tags: [
      { text: 'IEPL', tone: 'purple' },
      { text: 'IPv6', tone: 'green' },
    ],
    bandwidth: '800M',
    multiplier: '3x',
    latency: '-',
    load: '25.9M/s',
    loadTone: 'green',
    starred: false,
    selected: true,
  },
  {
    id: '5',
    flag: '🇹🇼',
    name: '台湾 I0',
    tags: [
      { text: 'AI', tone: 'gray' },
      { text: '动态IP', tone: 'blue' },
    ],
    bandwidth: '500M',
    multiplier: '2x',
    latency: '-',
    load: '550K/s',
    loadTone: 'green',
    starred: false,
  },
  {
    id: '6',
    flag: '🇯🇵',
    name: '日本 I0',
    tags: [
      { text: 'AWS', tone: 'pink' },
      { text: 'IPv6', tone: 'green' },
      { text: '动态IP', tone: 'blue' },
    ],
    bandwidth: '1G',
    multiplier: '1x',
    latency: '54ms',
    load: '2.1M/s',
    loadTone: 'green',
    starred: false,
    activeMark: true,
  },
])

function tagClass(tone: TagTone) {
  const map: Record<TagTone, string> = {
    pink: 'border border-pink-400 text-pink-600 bg-pink-50',
    green: 'border border-green-500 text-green-600 bg-green-50',
    purple: 'border border-purple-500 text-purple-600 bg-purple-50',
    gray: 'border border-gray-300 text-gray-600 bg-gray-50',
    blue: 'border border-blue-400 text-blue-600 bg-blue-50',
    orange: 'border border-orange-400 text-orange-600 bg-orange-50',
  }
  return map[tone]
}

function loadClass(tone: LoadTone) {
  if (tone === 'green')
    return 'text-#22c55e'
  if (tone === 'orange')
    return 'text-orange-500'
  return ''
}

function selectNode(node: ProxyNode) {
  nodeList.value.forEach((n) => {
    n.selected = n.id === node.id
  })
  currentNode.value = {
    flag: node.flag,
    name: node.name,
    connectedMark: !!node.activeMark,
  }
}

function toggleStar(node: ProxyNode) {
  node.starred = !node.starred
}
</script>
