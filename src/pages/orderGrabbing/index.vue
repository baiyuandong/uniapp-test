<template>
  <wd-navbar right-text="派单设置" :placeholder="true" safe-area-inset-top fixed @click-right="handleClickRight">
    <template #title>
      <text class="font-weight: 500; text-16px">
        导游抢单
      </text>
    </template>
    <template #right>
      <text class="font-weight: 500; text-14px">
        派单设置
      </text>
    </template>
  </wd-navbar>
  <!-- 订单列表 -->
  <scroll-view
    scroll-y class="h-[calc(100vh-336rpx)] bg-#f6f8fa py-3" :lower-threshold="20"
    @scrolltolower="loadMore"
  >
    <!-- 订单列表 -->
    <view class="mt-8rpx px-3">
      <view v-for="item in orderData.data" :key="item.id" class="mb-4 rounded-xl bg-white p-4 shadow-sm">
        <view @click="orderGrabbing(item)">
          <!-- 头部 -->
          <view class="mb-2 flex items-center justify-between">
            <view class="flex items-center space-x-2">
              <view
                class="h-6 w-6 flex items-center justify-center rounded bg-orange-500 text-xs text-white"
              >
                {{ item.demandInfoDO?.typeStr.substring(0, 1) }}
              </view>
              <text class="text-sm text-gray-800 font-medium">
                {{ item.demandInfoDO?.typeStr }}
              </text>
            </view>
            <text class="text-xs text-gray-400">
              订单编号：{{ item.orderCode }}
            </text>
          </view>

          <!-- 行程信息 -->
          <view class="flex flex-col text-xs text-gray-500 space-y-1">
            <text>出行日期：{{ item.demandInfoDO?.startTimeStr }}</text>
            <text>集合地：{{ item.demandInfoDO?.resort }}</text>
            <text>集合时间：{{ item.demandInfoDO?.resortTimeStr }}</text>
            <text v-if="item.demandInfoDO?.type == 1">
              出游景点：{{ item.demandInfoDO?.destination }}
            </text>
            <text v-if="item.demandInfoDO?.type == 2">
              出游景点：{{ item.demandInfoDO?.waypointNames }}
            </text>
            <text>导游等级：{{ item.demandInfoDO?.guideLevelName }}</text>
            <text>订单状态：{{ item.statusName }}</text>
          </view>

          <!-- 金额 -->
          <view v-if="item.price" class="mt-2 text-right text-base text-gray-800 font-medium">
            ¥{{ item.price }}
          </view>
        </view>

        <!-- 操作按钮 -->
        <view class="mt-3 flex justify-end space-x-2">
          <wd-button type="warning" size="small" class="rounded-full text-xs" @click="orderGrabbing(item)">
            抢单
          </wd-button>
        </view>
      </view>
    </view>
    <!-- 加载更多组件 -->
    <wd-loadmore
      :state="orderData.stateLoading" loading-text="正在加载数据..." finished-text="数据已全部加载完成"
      error-text="数据加载失败" @reload="loadMore"
    />
  </scroll-view>

  <wd-popup v-model="show" position="bottom" height="200px" :closable="true" @close="handleClose">
    <view class="ml-2 mt-24rpx text-left">
      派单状态设置
    </view>
    <wd-divider class="!mx-0 !px-0" />

    <view class="text-center text-gray-500">
      开启自动派单后，系统将自动为您匹配订单
    </view>
    <view class="p-24rpx px-60rpx py-34rpx">
      <wd-radio-group v-model="autoSendOrder" inline shape="dot" class="flex justify-between">
        <wd-radio value="0">
          开启自动派单
        </wd-radio>
        <wd-radio value="1">
          关闭自动派单
        </wd-radio>
      </wd-radio-group>
    </view>

    <view class="mb-24rpx mr-24rpx flex justify-end gap-4">
      <wd-button plain @click="handleClose">
        关 闭
      </wd-button>
      <wd-button @click="submitOrderConfig">
        确 定
      </wd-button>
    </view>
  </wd-popup>
</template>

<script lang="ts" setup>
import type { SocketConfig } from '@/utils/socket-io-manager'
import * as orderApi from '@/api/order'
import { useSocket } from '@/store/connectSocket'
import { useMessageStore } from '@/store/message'
import { useTokenStore } from '@/store/token'
import { SocketIOManager } from '@/utils/socket-io-manager'

defineOptions({
  name: 'TourGuideRobOrderIndex',
})

const messageStore = useMessageStore()

interface ChatMessage {
  content?: string
  sender?: string
  timestamp?: number
  isMine?: boolean
  type?: string
}
// Socket.IO 实例
const socketManager = ref<SocketIOManager | null>(null)
const tokenStore = useTokenStore()
const accessToken = tokenStore.getAccessToken()

const successData = ref()
// 响应式数据
const messages = ref<ChatMessage[]>([])
const inputMessage = ref('')
const isConnected = ref(false)
const socketId = ref('')

// Socket.IO 配置
const socketConfig: SocketConfig = {
  reconnection: true,
  reconnectionAttempts: 3,
  reconnectionDelay: 3000,
  auth: { authToken: `${accessToken}|WS` },
  query: {
    st: `${accessToken}|WS`,
  },
}

const orderData = ref({
  data: [],
  loading: false,
  finished: false,
  total: 0,
  pageNo: 1,
  pageSize: 10,
  status: undefined,
  stateLoading: undefined,
  loadingMore: false,
  refreshing: false,
  type: undefined,
})
// 派单状态
const autoSendOrder = ref(0)
definePage({
  priority: 0, // 优先级，数字越小优先级越高
  style: {
    navigationBarTitleText: '导游抢单',
    enablePullDownRefresh: true,
    navigationStyle: 'custom',
  },
})

const show = ref(false)

// 加载更多数据
function loadMore() {
  if (orderData.value.finished || orderData.value.loadingMore)
    return
  orderData.value.pageNo++
  sendMessage()
}

// 添加普通消息
function addMessage(message) {
  messages.value.push(message)
}

// 发送消息-切换需求类型或者修改分页数量
function sendMessage() {
  const message = {
    type: orderData.value.type,
    contentType: 0,
    messageId: '0',
    sessionId: '0',
    pageNo: orderData.value.pageNo,
    pageSize: 10,
  }

  // 添加到本地消息列表
  addMessage({
    ...message,
  })

  // 发送到服务器
  socketManager.value.emit('accept_user_message', message)

  // 清空输入框
  inputMessage.value = ''
}
// 派单设置
function handleClickRight() {
  show.value = true
}

// 关闭弹框
function handleClose() {
  show.value = false
}

// 保存抢单配置
function submitOrderConfig() {
  orderApi.updateDispatchOrderStatus({
    autoSendOrder: autoSendOrder.value,
  }).then((data) => {
    uni.showToast({
      title: '保存成功',
      icon: 'success',
    })

    show.value = false
  }).catch((error) => {
    uni.showToast({
      icon: 'error',
    })
  })
}

// 链接socket
function connect() {
  try {
    if (socketManager.value) {
      socketManager.value.destroy()
    }

    socketManager.value = new SocketIOManager(socketConfig)
    socketManager.value.initialize()

    // 监听系统事件
    socketManager.value.on('connect', (id: string) => {
      console.log('连接成功，ID:', id)
      isConnected.value = true
      socketId.value = id
    })

    socketManager.value.on('disconnect', (reason: string) => {
      console.log('断开连接:', reason)
      isConnected.value = false
      socketId.value = ''
    })

    socketManager.value.on('connect_error', (error: string) => {
      console.error('连接错误:', error)
    })

    socketManager.value.on('reconnect', (attempt: number) => {
      console.log('重连成功:', attempt)
    })

    // 监听自定义消息事件
    socketManager.value.on('system', (data: any) => {
      console.log('系统消息:', data)
      if (data && data.content) {
      }
    })

    socketManager.value.on('pong', (data: any) => {
      console.log('收到Pong响应:', data)
    })

    socketManager.value.on('send_room_message', (data: any) => {
      // 单独推送的内容
      const resData = JSON.parse(data)
      successData.value = data
      if (resData.code == 0) {
        const existingIds = new Set(orderData.value.data.map(item => item.id))
        const uniqueNewData = (resData?.detailOrderList ?? []).filter(item => !existingIds.has(item.id))
        orderData.value.data.push(...uniqueNewData)
        orderData.value.data.forEach((item) => {
          if (item.demandInfoDO.type == 2 && item.demandInfoDO.waypoint) {
            item.demandInfoDO.waypoint = JSON.parse(item.demandInfoDO.waypoint)
            item.demandInfoDO.waypointNames = item.demandInfoDO.waypoint.map(item => item.name).filter(name => name != '').join(',')
          }
        })
        // orderData.value.total = resData.totalData
        // orderData.value.finished = orderData.value.data.length >= orderData.value.total
        uni.stopPullDownRefresh()
        // orderData.value.stateLoading = orderData.value.finished ? 'finished' : 'loading'

        orderData.value.refreshing = false
      }
      else {
        uni.showToast({
          title: resData.errorMsg,
          icon: 'error',
        })
      }
    })

    // 监听订单消息
    socketManager.value.on('send_user_message', (data: any) => {
      const resData = JSON.parse(data)
      orderData.value.data.push(...resData.detailOrderList)
      orderData.value.data = Array.from(new Set(orderData.value.data))
      orderData.value.data.forEach((item) => {
        if (item.demandInfoDO.type == 2 && item.demandInfoDO.waypoint) {
          item.demandInfoDO.waypoint = JSON.parse(item.demandInfoDO.waypoint)
          item.demandInfoDO.waypointNames = item.demandInfoDO.waypoint.map(item => item.name).filter(name => name != '').join(',')
        }
      })
      orderData.value.total = resData.totalData
      orderData.value.finished = orderData.value.data.length >= orderData.value.total
      uni.stopPullDownRefresh()
      orderData.value.stateLoading = orderData.value.finished ? 'finished' : 'loading'
      orderData.value.refreshing = false
      orderData.value.loadingMore = false
      orderData.value.stateLoading = false
      console.log(orderData.value.data)
    })
  }
  catch (error) {
    console.error('Socket.IO 初始化失败:', error)
  }
}

// 获取 socket 链接
async function connectSocket() {
  try {
    socketManager.value = messageStore.getCachedSocketManager()

    // 如果缓存中没有 socketManager，则创建新连接
    if (!socketManager.value) {
      console.log('缓存中未找到 Socket 连接，开始创建新连接...')

      // 调用 useSocket 的 connect 方法并等待连接完成
      await new Promise<void>((resolve, reject) => {
        try {
          const { connect: connectFn, isConnected } = useSocket()
          connectFn()

          // 监听连接状态，连接成功后 resolve
          const checkConnection = setInterval(() => {
            if (isConnected.value) {
              clearInterval(checkConnection)
              socketManager.value = messageStore.getCachedSocketManager()
              console.log('Socket 连接成功')
              resolve()
            }
          }, 100)

          // 设置超时时间，避免无限等待
          setTimeout(() => {
            clearInterval(checkConnection)
            if (!socketManager.value) {
              reject(new Error('Socket 连接超时'))
            }
          }, 5000)
        }
        catch (error) {
          reject(error)
        }
      })
    }
    sendMessage()

    // 确保 socketManager.value 已赋值
    if (!socketManager.value) {
      uni.showToast({
        title: 'Socket 连接失败',
        icon: 'none',
      })
      return
    }

    socketManager.value.on('pong', (data: any) => {
      console.log('收到 Pong 响应:', data)
    })
    socketManager.value.on('send_room_message', (data: any) => {
      // 单独推送的内容
      const resData = JSON.parse(data)
      successData.value = data
      if (resData.code == 0) {
        const existingIds = new Set(orderData.value.data.map(item => item.id))
        const uniqueNewData = (resData?.detailOrderList ?? []).filter(item => !existingIds.has(item.id))
        orderData.value.data.push(...uniqueNewData)
        orderData.value.data.forEach((item) => {
          if (item.demandInfoDO.type == 2 && item.demandInfoDO.waypoint) {
            item.demandInfoDO.waypoint = JSON.parse(item.demandInfoDO.waypoint)
            item.demandInfoDO.waypointNames = item.demandInfoDO.waypoint.map(item => item.name).filter(name => name != '').join(',')
          }
        })
        // orderData.value.total = resData.totalData
        // orderData.value.finished = orderData.value.data.length >= orderData.value.total
        uni.stopPullDownRefresh()
        // orderData.value.stateLoading = orderData.value.finished ? 'finished' : 'loading'

        orderData.value.refreshing = false
      }
      else {
        uni.showToast({
          title: resData.errorMsg,
          icon: 'error',
        })
      }
    })

    // 监听订单消息
    socketManager.value.on('send_user_message', (data: any) => {
      const resData = JSON.parse(data)
      successData.value = data
      if (resData.code == 0) {
        const existingIds = new Set(orderData.value.data.map(item => item.id))
        const uniqueNewData = (resData?.detailOrderList ?? []).filter(item => !existingIds.has(item.id))
        orderData.value.data.push(...uniqueNewData)
        orderData.value.data.forEach((item) => {
          if (item.demandInfoDO.type == 2 && item.demandInfoDO.waypoint) {
            item.demandInfoDO.waypoint = JSON.parse(item.demandInfoDO.waypoint)
            item.demandInfoDO.waypointNames = item.demandInfoDO.waypoint.map(item => item.name).filter(name => name != '').join(',')
          }
        })
        orderData.value.total = resData.totalData
        orderData.value.finished = orderData.value.data.length >= orderData.value.total
        uni.stopPullDownRefresh()
        orderData.value.stateLoading = orderData.value.finished ? 'finished' : 'loading'
        orderData.value.refreshing = false
      }
      else {
        uni.showToast({
          title: resData.errorMsg,
          icon: 'error',
        })
      }
    })
  }
  catch (error) {
    console.error('Socket.IO 初始化失败:', error)
  }
}

// 抢单
function orderGrabbing(item) {
  uni.navigateTo({ url: `/pages/guide/detail?orderId=${item.id}` })
}

// 页面卸载时关闭连接
onUnload(() => {
  orderData.value.data = []
  if (socketManager.value) {
    // socketManager.value.destroy();
  }
})

// 页面隐藏的时候
onHide(() => {
  orderData.value.data = []
  if (socketManager.value) {
    // socketManager.value.destroy();
  }
})

onReady(() => {
  uni.pageScrollTo({
    scrollTop: 0,
    duration: 0, // 无动画
  })
})

onShow(() => {
  // 加载消息列表
  connect()
  // connectSocket()
})

onPullDownRefresh(() => {
  orderData.value.data = []
  orderData.value.pageNo = 1
  sendMessage()
})
</script>

<style lang="scss" scoped></style>
