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
                {{ item.demandInfoDO?.typeStr?.substring(0, 1) || '-' }}
              </view>
              <text class="text-sm text-gray-800 font-medium">
                {{ item.demandInfoDO?.typeStr || '--' }}
              </text>
            </view>
            <text class="text-xs text-gray-400">
              订单编号：{{ item.orderCode || '--' }}
            </text>
          </view>

          <!-- 行程信息 -->
          <view class="flex flex-col text-xs text-gray-500 space-y-1">
            <text>出行日期：{{ item.demandInfoDO?.startTimeStr || '--' }}</text>
            <text>集合地：{{ item.demandInfoDO?.resort || '--' }}</text>
            <text>集合时间：{{ item.demandInfoDO?.resortTimeStr || '--' }}</text>
            <text v-if="item.demandInfoDO?.type === 1">
              出游景点：{{ item.demandInfoDO?.destination || '--' }}
            </text>
            <text v-if="item.demandInfoDO?.type === 2">
              出游景点：{{ item.demandInfoDO?.waypointNames || '--' }}
            </text>
            <text>导游等级：{{ item.demandInfoDO?.guideLevelName || '--' }}</text>
            <text>订单状态：{{ item.statusName || '--' }}</text>
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
import { useTokenStore } from '@/store/token'
import { SocketIOManager } from '@/utils/socket-io-manager'

defineOptions({
  name: 'TourGuideRobOrderIndex',
})

interface ChatMessage {
  content?: string
  sender?: string
  timestamp?: number
  isMine?: boolean
  type?: string
}

interface WaypointItem {
  name?: string
}

interface DemandInfo {
  type?: number
  typeStr?: string
  startTimeStr?: string
  resort?: string
  resortTimeStr?: string
  destination?: string
  guideLevelName?: string
  waypoint?: string | WaypointItem[]
  waypointNames?: string
}

interface OrderItem {
  id: number | string
  orderCode?: string
  statusName?: string
  price?: number | string
  demandInfoDO?: DemandInfo
  [key: string]: any
}

interface SocketOrderPayload {
  code?: number
  errorMsg?: string
  totalData?: number
  detailOrderList?: OrderItem[]
}

const PAGE_SIZE = 10
const LOADMORE_LOADING_STATE = 'loading'
const LOADMORE_FINISHED_STATE = 'finished'
const LOADMORE_ERROR_STATE = 'error'

type LoadMoreState = typeof LOADMORE_LOADING_STATE | typeof LOADMORE_FINISHED_STATE | typeof LOADMORE_ERROR_STATE | ''
type RefreshSource = 'init' | 'refresh' | 'loadMore'
type SocketEventName = 'send_room_message' | 'send_user_message'

// Socket.IO 实例
const socketManager = ref<SocketIOManager | null>(null)
const tokenStore = useTokenStore()
const accessToken = tokenStore.getAccessToken()

const successData = ref<SocketOrderPayload | null>(null)
// 响应式数据
const messages = ref<ChatMessage[]>([])
const inputMessage = ref('')
const isConnected = ref(false)
const socketId = ref('')
const hasBoundListeners = ref(false)
const pendingRequestSource = ref<RefreshSource>('init')
const requestSequence = ref(0)

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
  data: [] as OrderItem[],
  loading: false,
  finished: false,
  total: 0,
  pageNo: 1,
  pageSize: PAGE_SIZE,
  status: undefined,
  stateLoading: '' as LoadMoreState,
  loadingMore: false,
  refreshing: false,
  type: undefined,
})
// 派单状态
const autoSendOrder = ref(0)
definePage({
  priority: 0,
  style: {
    navigationBarTitleText: '导游抢单',
    enablePullDownRefresh: true,
    navigationStyle: 'custom',
  },
})

const show = ref(false)

function safeParseSocketPayload(data: unknown): SocketOrderPayload | null {
  if (typeof data === 'string') {
    try {
      return JSON.parse(data)
    }
    catch (error) {
      console.error('解析 socket 数据失败:', error, data)
      return null
    }
  }

  if (typeof data === 'object' && data !== null)
    return data as SocketOrderPayload

  return null
}

function parseWaypoint(waypoint: DemandInfo['waypoint']): WaypointItem[] {
  if (Array.isArray(waypoint))
    return waypoint

  if (!waypoint || typeof waypoint !== 'string')
    return []

  try {
    const parsed = JSON.parse(waypoint)
    return Array.isArray(parsed) ? parsed : []
  }
  catch (error) {
    console.error('解析 waypoint 失败:', error, waypoint)
    return []
  }
}

function normalizeOrderItem(item: OrderItem): OrderItem {
  const demandInfo = item?.demandInfoDO
  if (!demandInfo)
    return item

  const waypointList = parseWaypoint(demandInfo.waypoint)
  const waypointNames = waypointList
    .map(waypoint => waypoint?.name?.trim())
    .filter(Boolean)
    .join(',')

  return {
    ...item,
    demandInfoDO: {
      ...demandInfo,
      waypoint: waypointList,
      waypointNames: demandInfo.type === 2 ? waypointNames : demandInfo.waypointNames,
    },
  }
}

function mergeOrders(currentOrders: OrderItem[], incomingOrders: OrderItem[], replace = false): OrderItem[] {
  const orderMap = new Map<OrderItem['id'], OrderItem>()
  const baseOrders = replace ? [] : currentOrders

  baseOrders.forEach((item) => {
    if (item?.id !== undefined && item?.id !== null)
      orderMap.set(item.id, item)
  })

  incomingOrders.forEach((item) => {
    const normalizedItem = normalizeOrderItem(item)
    if (normalizedItem?.id === undefined || normalizedItem?.id === null)
      return

    const existingItem = orderMap.get(normalizedItem.id)
    orderMap.set(normalizedItem.id, existingItem ? { ...existingItem, ...normalizedItem } : normalizedItem)
  })

  return Array.from(orderMap.values())
}

function finishRequest() {
  orderData.value.loading = false
  orderData.value.loadingMore = false
  orderData.value.refreshing = false
  uni.stopPullDownRefresh()
}

function setLoadMoreState(state?: LoadMoreState) {
  orderData.value.stateLoading = state ?? ''
}

function syncFinishedState() {
  const hasReachedTotal = orderData.value.total > 0 && orderData.value.data.length >= orderData.value.total
  const pageNotFull = !orderData.value.loading && orderData.value.data.length > 0 && orderData.value.data.length % orderData.value.pageSize !== 0
  orderData.value.finished = hasReachedTotal || pageNotFull
  setLoadMoreState(orderData.value.finished ? LOADMORE_FINISHED_STATE : '')
}

function resetOrderList() {
  orderData.value.data = []
  orderData.value.pageNo = 1
  orderData.value.total = 0
  orderData.value.finished = false
  orderData.value.loading = false
  orderData.value.loadingMore = false
  orderData.value.refreshing = false
  setLoadMoreState('')
}

// 加载更多数据
function loadMore() {
  if (!isConnected.value || orderData.value.finished || orderData.value.loading || orderData.value.loadingMore)
    return

  orderData.value.pageNo += 1
  orderData.value.loadingMore = true
  setLoadMoreState(LOADMORE_LOADING_STATE)
  requestOrders('loadMore')
}

// 添加普通消息
function addMessage(message: ChatMessage) {
  messages.value.push(message)
}

function buildOrderRequestMessage() {
  return {
    type: orderData.value.type,
    contentType: 0,
    messageId: `${Date.now()}_${requestSequence.value}`,
    sessionId: '0',
    pageNo: orderData.value.pageNo,
    pageSize: orderData.value.pageSize,
  }
}

// 发送消息-切换需求类型或者修改分页数量
function requestOrders(source: RefreshSource = 'init') {
  if (!socketManager.value || !isConnected.value) {
    console.warn('socket 未连接，跳过请求')
    finishRequest()
    if (source === 'loadMore' && orderData.value.pageNo > 1)
      orderData.value.pageNo -= 1
    setLoadMoreState(LOADMORE_ERROR_STATE)
    return
  }

  pendingRequestSource.value = source
  requestSequence.value += 1

  if (source === 'refresh') {
    orderData.value.refreshing = true
    orderData.value.loadingMore = false
  }
  else if (source === 'loadMore') {
    orderData.value.loadingMore = true
  }
  else {
    orderData.value.loading = true
  }

  const message = buildOrderRequestMessage()

  addMessage({
    ...message,
    timestamp: Date.now(),
  })

  socketManager.value.emit('accept_user_message', message)
  inputMessage.value = ''
}

function handleSocketError(errorMsg = '数据加载失败', source = pendingRequestSource.value) {
  if (source === 'loadMore' && orderData.value.pageNo > 1)
    orderData.value.pageNo -= 1

  finishRequest()
  setLoadMoreState(LOADMORE_ERROR_STATE)
  uni.showToast({
    title: errorMsg,
    icon: 'none',
  })
}

function applySocketOrders(payload: SocketOrderPayload, eventName: SocketEventName) {
  successData.value = payload

  if (payload.code !== 0) {
    handleSocketError(payload.errorMsg || '订单数据异常')
    return
  }

  const source = pendingRequestSource.value
  const incomingOrders = Array.isArray(payload.detailOrderList) ? payload.detailOrderList : []
  const shouldReplace = eventName === 'send_user_message' && source === 'refresh'

  orderData.value.data = mergeOrders(orderData.value.data, incomingOrders, shouldReplace)

  if (eventName === 'send_user_message') {
    orderData.value.total = Number(payload.totalData || 0)
    if (incomingOrders.length < orderData.value.pageSize)
      orderData.value.finished = true
    else
      syncFinishedState()
  }
  else {
    syncFinishedState()
  }

  finishRequest()
  if (!orderData.value.finished)
    setLoadMoreState('')
}

function registerSocketListeners() {
  if (!socketManager.value || hasBoundListeners.value)
    return

  socketManager.value.on('connect', (id: string) => {
    console.log('连接成功，ID:', id)
    isConnected.value = true
    socketId.value = id
    requestOrders(orderData.value.data.length ? 'refresh' : 'init')
  })

  socketManager.value.on('disconnect', (reason: string) => {
    console.log('断开连接:', reason)
    isConnected.value = false
    socketId.value = ''
    finishRequest()
  })

  socketManager.value.on('connect_error', (error: string) => {
    console.error('连接错误:', error)
    handleSocketError('Socket 连接错误')
  })

  socketManager.value.on('reconnect', (attempt: number) => {
    console.log('重连成功:', attempt)
    isConnected.value = true
    requestOrders('refresh')
  })

  socketManager.value.on('system', (data: any) => {
    console.log('系统消息:', data)
  })

  socketManager.value.on('pong', (data: any) => {
    console.log('收到Pong响应:', data)
  })

  socketManager.value.on('send_room_message', (data: unknown) => {
    const payload = safeParseSocketPayload(data)
    if (!payload) {
      handleSocketError('推送数据解析失败')
      return
    }
    applySocketOrders(payload, 'send_room_message')
  })

  socketManager.value.on('send_user_message', (data: unknown) => {
    const payload = safeParseSocketPayload(data)
    if (!payload) {
      handleSocketError('订单数据解析失败')
      return
    }
    applySocketOrders(payload, 'send_user_message')
  })

  hasBoundListeners.value = true
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
  }).then(() => {
    uni.showToast({
      title: '保存成功',
      icon: 'success',
    })

    show.value = false
  }).catch((error) => {
    console.error('保存抢单配置失败:', error)
    uni.showToast({
      title: '保存失败',
      icon: 'none',
    })
  })
}

// 链接socket
function connect() {
  try {
    if (socketManager.value) {
      socketManager.value.destroy()
      hasBoundListeners.value = false
    }

    socketManager.value = new SocketIOManager(socketConfig)
    registerSocketListeners()
    socketManager.value.initialize()
  }
  catch (error) {
    console.error('Socket.IO 初始化失败:', error)
    handleSocketError('Socket 初始化失败')
  }
}

// 抢单
function orderGrabbing(item: OrderItem) {
  uni.navigateTo({ url: `/pages/guide/detail?orderId=${item.id}` })
}

function clearOrderPageState() {
  resetOrderList()
  messages.value = []
  inputMessage.value = ''
}

// 页面卸载时关闭连接
onUnload(() => {
  clearOrderPageState()
})

// 页面隐藏的时候
onHide(() => {
  clearOrderPageState()
})

onReady(() => {
  uni.pageScrollTo({
    scrollTop: 0,
    duration: 0,
  })
})

onShow(() => {
  connect()
})

onPullDownRefresh(() => {
  resetOrderList()
  requestOrders('refresh')
})
</script>

<style lang="scss" scoped></style>
