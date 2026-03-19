<template>
  <!-- <wd-navbar fixed safeAreaInsetTop placeholder class="bg-white" id="navBarAreaBox" @click-left="goback">
		<template #left>
			<wd-icon class="icon-left" name="thin-arrow-left" size="18" color="#000" />
		</template>
		<template #right>
			<wd-drop-menu :modal="false">
				<wd-drop-menu-item title="" icon="ellipsis" icon-size="16px">
					<wd-button type="text" @click="applicationCustomerService">申请客服介入</wd-button>
				</wd-drop-menu-item>
			</wd-drop-menu>
		</template>
		<template #title>
			<wd-navbar title="聊天" id="navbarBox"></wd-navbar>
		</template>
	</wd-navbar> -->
  <view class="chatContainer w-100% bg-[#EDEDED]">
    <view class="pb-chat-container flex flex-col justify-between">
      <view class="chat-content">
        <!-- 聊天区域 -->
        <scroll-view
          id="scrollview" class="chat-content-box bg-[#F6F6F6]" :scroll-y="true"
          :scroll-top="scrollTop" :style="chatContentStyle" @scroll="onScroll"
        >
          <view id="msglistview" class="chat-body">
            <!-- 加载提示 -->
            <view v-if="loadingHistory" class="loading-hint py-20rpx text-center text-24rpx text-gray-500">
              正在加载历史记录...
            </view>
            <view
              v-if="!hasMoreHistory && msgList.length > 0"
              class="no-more-hint py-20rpx text-center text-24rpx text-gray-400"
            >
              已加载全部历史记录
            </view>

            <view v-for="(item, index) in msgList" :key="index">
              <!-- {{ item }} -->
              <!-- 自己发的消息 -->
              <view v-if="item.type == 0 && item.childrenType == 0">
                <view v-if="item.userContent != ''" class="item self">
                  <view class="content right" v-html="item.userContent" />
                  <image v-if="!item.image" class="avatar" src="/static/images/msg/defaultIcon.png" />
                  <image v-else class="avatar" :src="item.image" />
                </view>
                <!-- 机器人发的消息 -->
                <view v-if="item.content != ''" class="item Ai">
                  <image v-if="!item.image" class="avatar" src="/static/images/msg/defaultIcon.png" />
                  <image v-else class="avatar" :src="item.image" />
                  <view class="content left" v-html="item.content" />
                </view>
              </view>
              <view v-if="item.type == 5" class="text-center text-24rpx">
                {{ item.content }}
              </view>

              <!-- 图片消息 -->
              <view v-if="item.type == 0 && item.childrenType == 1">
                <view v-if="item.userContent != ''" class="item self">
                  <view class="content right image-message">
                    <image
                      :src="item.content || item.userContent" mode="widthFix"
                      class="h-100px w-100px"
                    />
                  </view>
                  <image v-if="!item.image" class="avatar" src="/static/images/msg/defaultIcon.png" />
                  <image v-else class="avatar" :src="item.image" />
                </view>
                <view v-if="item.content != ''" class="item Ai">
                  <image v-if="!item.image" class="avatar" src="/static/images/msg/defaultIcon.png" />
                  <image v-else class="avatar" :src="item.image" />
                  <view class="content left image-message">
                    <image :src="item.content" mode="widthFix" class="h-100px w-100px" />
                  </view>
                </view>
              </view>
            </view>
            <view id="chatBottomAnchor" class="chat-bottom-anchor" />
          </view>
        </scroll-view>
      </view>
      <view class="chat-footer" :style="chatFooterStyle">
        <view class="chat-bottom">
          <view class="send-msg">
            <view class="uni-textarea">
              <wd-textarea
                ref="chatInputRef" v-model="htmlInput" class="chat-input-box" :maxlength="300"
                confirm-type="send" placeholder="请输入内容..." :disable-default-padding="true"
                :show-confirm-bar="false" :adjust-position="false" auto-height
                :no-border="true" @confirm="handleSend" @linechange="sendHeight" @focus="focus" @blur="blur"
              />
            </view>
            <wd-button type="icon" icon="chat" @click="openEmojiPanel" />
            <wd-button type="icon" icon="add" @click="openTool" />
            <wd-button size="small" type="text" class="send-btn" @click="handleSend">
              发送
            </wd-button>
          </view>
        </view>
        <!-- 表情面板 -->
        <view v-if="showEmojiPanel" class="h-44 pl-30rpx pr-30rpx">
          <scroll-view scroll-y class="h-44">
            <view class="grid grid-cols-8 gap-3">
              <view
                v-for="(item, index) in emojiList" :key="index"
                class="h-8 w-8 flex items-center justify-center text-xl transition-transform active:scale-110"
                @tap="addEmoji(item)"
              >
                <text>{{ item }}</text>
              </view>
            </view>
          </scroll-view>
        </view>

        <!-- 工具面板 -->
        <view v-if="showTool" class="h-44 pl-30rpx pr-30rpx">
          <scroll-view scroll-y class="h-44">
            <view class="grid grid-cols-4 gap-3">
              <view v-for="(icon, index) in iconsList" :key="index" class="icon-item">
                <wd-upload
                  :file-list="fileList" action="/dst/service/file/v1/storage/upload"
                  :header="uploadHeader" @change="handleChange"
                >
                  <text class="icon">
                    {{ icon.emoji }}
                  </text>
                  <text class="icon-label">
                    {{ icon.label }}
                  </text>
                  <template #file-list />
                </wd-upload>
              </view>
            </view>
          </scroll-view>
        </view>
      </view>
    </view>
  </view>
</template>

<script lang="ts" setup>
import type { SocketConfig } from '@/utils/socket-io-manager'
import { createChat, getChatPage } from '@/api/message'
import { useSocket } from '@/store/connectSocket'
import { useMessageStore } from '@/store/message'
import { useTokenStore } from '@/store/token'
import { useUserStore } from '@/store/user'
import { SocketIOManager } from '@/utils/socket-io-manager'

defineOptions({
  name: 'MessageChatPage',
})
const tokenStore = useTokenStore()
const accessToken = tokenStore.getAccessToken()
const messageStore = useMessageStore()

const showEmojiPanel = ref<boolean>(false)
const chatInputRef = ref<HTMLElement | null>(null) // 添加输入框引用
const isInputFocused = ref(false) // 输入框聚焦状态

interface ChatMessage {
  content?: string
  sender?: string
  timestamp?: number
  isMine?: boolean
  type?: string
}

const value1 = ref<number>(0)
const option1 = ref<Record<string, any>[]>([
  { label: '申请客服介入', value: 0 },
])

const emojiList = ref<string[]>([
  '😀',
  '😃',
  '😄',
  '😁',
  '😆',
  '😅',
  '😂',
  '🤣',
  '😊',
  '😇',
  '🙂',
  '🙃',
  '😉',
  '😌',
  '😍',
  '🥰',
  '😘',
  '😗',
  '😙',
  '😚',
  '😋',
  '😛',
  '😝',
  '😜',
  '🤪',
  '🤨',
  '🧐',
  '🤓',
  '😎',
  '🤩',
  '🥳',
  '😏',
  '😒',
  '😞',
  '😔',
  '😟',
  '😕',
  '🙁',
  '☹️',
  '😣',
  '😖',
  '😫',
  '😩',
  '🥺',
  '😢',
  '😭',
  '😤',
  '😠',
  '😡',
  '🤬',
  '🤯',
  '😳',
  '🥵',
  '🥶',
  '😱',
  '😨',
  '😰',
  '😥',
  '😓',
  '🤗',
  '🤔',
  '🤭',
  '🤫',
  '🤥',
  '😶',
  '😐',
  '😑',
  '😬',
  '🙄',
  '😯',
  '😦',
  '😧',
  '😮',
  '😲',
  '🥱',
  '😴',
  '🤤',
  '😪',
  '😵',
  '🤐',
  '🥴',
  '🤢',
  '🤮',
  '🤧',
  '😷',
  '🤒',
  '🤕',
  '🤑',
  '🤠',
  '😈',
  '👿',
  '👹',
  '👺',
  '🤡',
  '💩',
  '👻',
  '💀',
  '☠️',
  '👽',
  '👾',
  '🤖',
  '🎃',
  '😺',
  '😸',
  '😹',
  '😻',
  '😼',
  '😽',
  '🙀',
  '😿',
  '😾',
])

definePage({
  style: {
    // navigationStyle: 'custom',
    navigationBarTitleText: '聊天',
  },
})

const uploadHeader = {
  'Authorization': `Bearer ${accessToken}`,
  'Sign-in': import.meta.env.VITE_CLIENT_TOKEN_KEY,
}

const socketManager = ref<SocketIOManager | null>(null)
const showTool = ref(false)
const fileList = ref([])

// 响应式数据
const messages = ref<ChatMessage[]>([])
const isConnected = ref(false)
const socketId = ref('')
const userStore = useUserStore()
const receiveUserId = ref()
const editorRef = ref()

const loadingHistory = ref(false) // 加载状态
const hasMoreHistory = ref(true) // 是否还有更多历史记录
const isFirstLoad = ref(true) // 是否是首次加载
const historyQueryParams = reactive({
  pageNum: 0,
  pageSize: 50,

})

const iconsList = ref([
  { id: 1, emoji: '📷', label: '相机', action: 'camera' },
  // { id: 2, emoji: '🖼️', label: '相册', action: 'album' },
])

// 替换 chatMsg 为两个变量
const rawInput = ref('') // 纯文本内容
const htmlInput = ref('') // HTML 内容

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

// 打开工具
function hideInputKeyboard() {
  uni.hideKeyboard()
}

// 打开工具
function openTool() {
  hideInputKeyboard()
  showEmojiPanel.value = false // 关闭图标
  showTool.value = !showTool.value
}

// 打开图标
function openEmojiPanel() {
  hideInputKeyboard()
  showTool.value = false
  showEmojiPanel.value = !showEmojiPanel.value
}

// 消息id
const messageId = ref('')
onLoad((options) => {
  messageId.value = options.ids
})

onMounted(() => {
  getUserMessageListEvent()
  uni.onKeyboardHeightChange(({ height }) => {
    keyboardHeight.value = height - 20
    nextTick(() => {
      updateChatFooterHeight()
      scrollToBottom()
    })
  })
  setTimeout(() => {
    // 给内容区域计算高度
    setContentAreaHeightEvent()
    // 初始化底部区域高度
    updateChatFooterHeight()
    // 定位到消息最底部
    scrollToBottom()
  })
})

// 消息列表数据
const msgList = ref([])
const asUserList = ref([])
const sessionType = ref()
const historyMsgTotal = ref(0)

// 底部聊天区域高度
const keyboardHeight = ref(0)
const chatContentStyle = computed(() => ({
  paddingBottom: `${chatFooterHeight.value + keyboardHeight.value}px`,
}))
const chatFooterStyle = computed(() => ({
  transform: keyboardHeight.value > 0 ? `translateY(-${keyboardHeight.value}px)` : 'translateY(0)',
}))

function getUserMessageListEvent() {
  msgList.value = []
}

// 给内容区域计算高度
const contentInfoHeight = ref(0)
// 聊天内容区域高度（减去头部和底部）
const chatContentHeight = ref(0)
// 底部聊天区域高度
const chatFooterHeight = ref(0)
function setContentAreaHeightEvent() {
  uni.getSystemInfo({
    success(res) {
      const screenHeight = res.screenHeight
      // 获取头部导航栏高度
      uni.createSelectorQuery().select('#navBarAreaBox').boundingClientRect((data) => {
        if (data && 'height' in data) {
          const navBarHeight = data.height
          const remainingHeight = screenHeight - navBarHeight
          contentInfoHeight.value = remainingHeight

          // 获取底部聊天区域高度
          uni.createSelectorQuery().select('.chat-footer').boundingClientRect((footerData) => {
            if (footerData && 'height' in footerData) {
              chatFooterHeight.value = footerData.height
              // 计算聊天内容区域高度：总剩余高度 - 底部区域高度
              chatContentHeight.value = remainingHeight - footerData.height
            }
          }).exec()
        }
      }).exec()
    },
  })
}

// 监听底部区域高度变化
watch(chatFooterHeight, (newHeight) => {
  if (newHeight > 0 && contentInfoHeight.value > 0) {
    chatContentHeight.value = contentInfoHeight.value - newHeight
  }
})

// 文件上传成功
function handleChange(file) {
  const response = JSON.parse(file.fileList[0].response)
  fileList.value = []
  sendImageMessage(response.data?.presUrl, response.data?.presUrl)
}

// 申请客服
function applicationCustomerService() {
  socketManager.value.emit('please_customer', { sessionId: receiveUserId.value })
}

// 发送图片消息
function sendImageMessage(imageUrl: string, localPath: string) {
  // 先添加到本地消息列表显示
  const imageMsg = {
    type: 0, // 图片消息类型
    content: '', // 如果是接收的消息
    userContent: imageUrl, // 自己发送的图片URL
    image: userStore.userInfo.avatarLogo,
    messageType: 'image', // 自定义字段，标识消息类型
    localPath: imageUrl, // 本地路径，用于显示临时图片
    timestamp: Date.now(),
    childrenType: '1', // 1 图片消息
  }

  // 添加到消息列表
  msgList.value.push(imageMsg)

  // 滚动到底部
  setTimeout(() => {
    scrollToBottom()
  }, 100)

  // 发送到服务器
  const message = {
    type: 0, // 图片消息类型
    sendUserId: userStore.userInfo.id,
    receiveUserId: receiveUserId.value,
    content: imageUrl, // 图片URL
    sessionType: sessionType.value,
    sessionId: receiveUserId.value,
    childrenType: '1', // 1 图片消息
  }

  // 通过WebSocket发送
  if (socketManager.value) {
    socketManager.value.emit('message_event', message)
    console.log('已发送图片消息到服务器:', message)
  }
  else {
    console.error('WebSocket未连接，无法发送图片消息')
    uni.showToast({
      title: '发送失败，请检查网络连接',
      icon: 'none',
    })
  }
}

// 返回
function goback() {
  uni.navigateBack()
}

function focus() {
  scrollToBottom()
}

function blur() {
  scrollToBottom()
}

function rpxTopx(px) {
  let deviceWidth = uni.getSystemInfoSync().windowWidth
  let rpx = (750 / deviceWidth) * Number(px)
  return Math.floor(rpx)
}

const bottomHeight = ref(0)
function sendHeight() {
  setTimeout(() => {
    let query = uni.createSelectorQuery()
    query.select('.send-msg').boundingClientRect()
    query.exec((res) => {
      if (res[0] && 'height' in res[0]) {
        bottomHeight.value = rpxTopx(res[0].height)
        // 更新底部聊天区域总高度
        updateChatFooterHeight()
      }
    })
  }, 10)
}

// 更新底部聊天区域高度
function updateChatFooterHeight() {
  uni.createSelectorQuery().select('.chat-footer').boundingClientRect((data) => {
    if (data && 'height' in data) {
      chatFooterHeight.value = data.height
    }
  }).exec()
}

// 监听表情面板开关状态变化
watch(showEmojiPanel, (isOpen) => {
  if (isOpen) {
    // 表情面板打开时，延迟更新高度
    setTimeout(() => {
      updateChatFooterHeight()
    }, 100)
  }
  else {
    // 表情面板关闭时，立即更新高度
    nextTick(() => {
      updateChatFooterHeight()
    })
  }
})

// 监听表情面板开关状态变化
watch(showTool, (isOpen) => {
  if (isOpen) {
    // 表情面板打开时，延迟更新高度
    setTimeout(() => {
      updateChatFooterHeight()
    }, 100)
  }
  else {
    // 表情面板关闭时，立即更新高度
    nextTick(() => {
      updateChatFooterHeight()
    })
  }
})

// 点击发送要显示按键板
const scrollTop = ref(0)
const scrollIntoView = ref('')
function scrollToBottom() {
  nextTick(() => {
    scrollIntoView.value = 'chatBottomAnchor'
    setTimeout(() => {
      scrollIntoView.value = ''
      uni.createSelectorQuery().select('#scrollview').boundingClientRect().select('#msglistview').boundingClientRect().exec((res) => {
        if (res[0] && res[1] && res[1].height > res[0].height) {
          scrollTop.value = rpxTopx(res[1].height - res[0].height)
        }
      })
    }, 30)
  })
}

// 自动调整输入框高度
function autoResize() {
  if (!chatInputRef.value)
    return

  // 重置高度
  chatInputRef.value.style.height = 'auto'

  // 计算内容高度
  const scrollHeight = chatInputRef.value.scrollHeight
  const maxHeight = 500 // 500rpx = 250px

  // 设置高度，不超过最大高度
  const newHeight = Math.min(scrollHeight, maxHeight)
  chatInputRef.value.style.height = `${newHeight}px`

  // 更新底部区域高度
  sendHeight()
}

// 发送
async function handleSend() {
  // const msg = await editorRef.value.getContents()
  const msg = htmlInput.value
  if (!msg) {
    // 空消息不发送
    return
  }

  let obj = {
    type: 0,
    content: '',
    userContent: msg, // 发送纯文本
    image: userStore.userInfo.avatarLogo,
    childrenType: 0,
  }
  msgList.value.push(obj)

  try {
    await sendMessage(msg)
  }
  catch (error) {
    console.error('Error adding message:', error)
  }

  // 清空输入框
  if (chatInputRef.value) {
    chatInputRef.value.innerHTML = ''
    rawInput.value = ''
    htmlInput.value = ''
    chatInputRef.value.style.height = 'auto'
    // 更新底部区域高度
    sendHeight()
  }
  htmlInput.value = ''

  scrollToBottom()
}

// 添加表情到输入框
function addEmoji(emojiItem) {
  // editorRef.value.insertFace(emojiItem.title)

  htmlInput.value = htmlInput.value + emojiItem

  // // 隐藏表情面板
  showEmojiPanel.value = false
  // 自动调整高度
  autoResize()
}

// 添加普通消息
function addMessage(message) {
  messages.value.push(message)
}

// 链接 socket
async function connect() {
  try {
    socketManager.value = messageStore.getCachedSocketManager()

    // 如果缓存中没有 socketManager，则创建新连接
    if (!socketManager.value) {
      console.log('缓存中未找到 Socket 连接，开始创建新连接...')

      // 调用 useSocket 的 connect 方法并等待连接完成
      await new Promise<void>((resolve, reject) => {
        try {
          const { connect: connectSocket, isConnected } = useSocket()
          connectSocket()

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

    // 确保 socketManager.value 已赋值
    if (!socketManager.value) {
      uni.showToast({
        title: 'Socket 连接失败',
        icon: 'none',
      })
      return
    }

    if (isFirstLoad.value) {
      queryHistoryMsg(true)
      isFirstLoad.value = false
    }

    // 监听自定义消息事件
    socketManager.value.on('system', (data: any) => {
      console.log('系统消息:', data)
      if (data && data.content) {
      }
    })

    // 监听订单消息
    socketManager.value.on('user_event_message', (data: any) => {
      // 聊天历史记录
      if (data.type === 9) {
        uni.setStorageSync(receiveUserId.value, data.messageList)
      }
      if (data.type === 8) {
        // 将未读写入store
        // if (!data.messageList && !store.conversitionList) return
        // // 转换json
        historyMsgTotal.value = data.num
        const messageList = Array.isArray(data.messageList) ? data.messageList : []

        if (messageList.length === 0) {
          // 没有更多历史记录
          hasMoreHistory.value = false
          loadingHistory.value = false
          return
        }

        const reversedMessageList = [...messageList].reverse()
        // 循环
        reversedMessageList.forEach((item: any) => {
          const matchedUser = asUserList.value.find(user => item.sendUserId === user.id)
          if (item.sendUserId == userStore.userInfo.id) {
            item.image = userStore.userInfo.avatarLogo
            item.userContent = item.content
            item.content = ''
          }
          else {
            item.image = matchedUser ? matchedUser.avatarLogo : ''
            item.userContent = ''
          }
        })

        // 如果是第一页，直接替换
        if (historyQueryParams.pageNum === 0) {
          msgList.value = reversedMessageList
        }
        else {
          // 否则添加到开头
          msgList.value.unshift(...reversedMessageList)
        }

        // 如果是首次加载，滚动到底部
        if (historyQueryParams.pageNum === 0) {
          setTimeout(() => {
            scrollToBottom()
          }, 100)
        }
        else {
          // 如果是加载更多，保持当前位置
          // keepScrollPosition(msgList.value.length);
        }

        // 更新页码
        if (messageList.length < historyQueryParams.pageSize) {
          hasMoreHistory.value = false
        }
        else {
          historyQueryParams.pageNum += 1
        }
        loadingHistory.value = false
      }
      // 已读消息
      if (data.type === 7) {
        // 创建会话
        // if (!store.unRead) return
        // let num = 0
        // store.unRead.messageList.forEach((unItem: any) => {
        // 	if (unItem.id == store.sessionSelectId) {
        // 		num = unItem.num
        // 		unItem.num = 0
        // 	}
        // })
        // store.unRead.num = -num
      }
      // 未读消息
      if (data.type === 6) {
        // 将未读写入store
        // store.unRead = data
      }

      // 申请客服介入消息
      if (data.type === 5) {
      }

      // 错误消息
      if (data.type === 4) {
      }

      // 客服消息
      if (data.type === 3) {
      }

      // 系统消息
      if (data.type === 2) {
      }

      // 群组消息
      if (data.type === 1) {
      }

      // 用户消息
      if (data.type === 0) {
        // 转换data
        if (!data.messageList)
          return
        data.messageList.forEach((item) => {
          const matchedUser = asUserList.value.find(user => item.sendUserId === user.id)
          item.image = matchedUser ? matchedUser.avatarLogo : ''
          item.userContent = ''
          item.type = 0
        })

        msgList.value.push(...Array.isArray(data.messageList) ? data.messageList : [])
        scrollToBottom()
      }
    })
  }
  catch (error) {
    console.error('Socket.IO 初始化失败:', error)
  }
}

function connect2() {
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
      // 连接成功后立即加载历史记录
      // messageStore.setSocketManager(socketManager.value as SocketIOManager)

      if (isFirstLoad.value) {
        queryHistoryMsg(true)
        isFirstLoad.value = false
      }
    })

    socketManager.value.on('disconnect', (reason: string) => {
      console.log('断开连接:', reason)
      isConnected.value = false
      socketId.value = ''
      // 断开连接时清除缓存
      messageStore.clearCachedSocketManager()
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

    // 监听订单消息
    socketManager.value.on('user_event_message', (data: any) => {
      // 聊天历史记录
      if (data.type === 9) {
        uni.setStorageSync(receiveUserId.value, data.messageList)
      }
      if (data.type === 8) {
        // 将未读写入store
        // if (!data.messageList && !store.conversitionList) return
        // // 转换json
        historyMsgTotal.value = data.num
        const messageList = Array.isArray(data.messageList) ? data.messageList : []
        if (messageList.length === 0) {
          // 没有更多历史记录
          hasMoreHistory.value = false
          loadingHistory.value = false
          return
        }

        const reversedMessageList = [...messageList].reverse()
        // 循环
        reversedMessageList.forEach((item: any) => {
          const matchedUser = asUserList.value.find(user => item.sendUserId === user.id)
          if (item.sendUserId == userStore.userInfo.id) {
            item.image = userStore.userInfo.avatarLogo
            item.userContent = item.content
            item.content = ''
          }
          else {
            item.image = matchedUser ? matchedUser.avatarLogo : ''
            item.userContent = ''
          }
        })

        // 如果是第一页，直接替换
        if (historyQueryParams.pageNum === 0) {
          msgList.value = reversedMessageList
        }
        else {
          // 否则添加到开头
          msgList.value.unshift(...reversedMessageList)
        }

        // 如果是首次加载，滚动到底部
        if (historyQueryParams.pageNum === 0) {
          setTimeout(() => {
            scrollToBottom()
          }, 100)
        }
        else {
          // 如果是加载更多，保持当前位置
          // keepScrollPosition(msgList.value.length);
        }

        // 更新页码
        if (messageList.length < historyQueryParams.pageSize) {
          hasMoreHistory.value = false
        }
        else {
          historyQueryParams.pageNum += 1
        }
        loadingHistory.value = false
      }
      // 已读消息
      if (data.type === 7) {
        // 创建会话
        // if (!store.unRead) return
        // let num = 0
        // store.unRead.messageList.forEach((unItem: any) => {
        // 	if (unItem.id == store.sessionSelectId) {
        // 		num = unItem.num
        // 		unItem.num = 0
        // 	}
        // })
        // store.unRead.num = -num
      }
      // 未读消息
      if (data.type === 6) {
        // 将未读写入store
        // store.unRead = data
      }

      // 申请客服介入消息
      if (data.type === 5) {
      }

      // 错误消息
      if (data.type === 4) {
      }

      // 客服消息
      if (data.type === 3) {
      }

      // 系统消息
      if (data.type === 2) {
      }

      // 群组消息
      if (data.type === 1) {
      }

      // 用户消息
      if (data.type === 0) {
        // 转换data
        if (!data.messageList)
          return
        data.messageList.forEach((item) => {
          const matchedUser = asUserList.value.find(user => item.sendUserId === user.id)
          item.image = matchedUser ? matchedUser.avatarLogo : ''
          item.userContent = ''
          item.type = 0
        })

        msgList.value.push(...Array.isArray(data.messageList) ? data.messageList : [])
        scrollToBottom()
      }
    })
  }
  catch (error) {
    console.error('Socket.IO 初始化失败:', error)
  }
}

// 发送消息-切换需求类型或者修改分页数量
function sendMessage(msg) {
  // createConversation()
  console.log(userStore.userInfo.id)
  console.log(receiveUserId.value)
  const message = {
    type: 0,
    sendUserId: userStore.userInfo.id,
    receiveUserId: receiveUserId.value,
    content: msg, // 改为发送纯文本
    sessionType: sessionType.value,
    sessionId: receiveUserId.value,
    childrenType: '0', // 0 文本消息  1 图片消息
  }

  // 添加到本地消息列表
  addMessage({
    ...message,
  })

  // 发送到服务器
  socketManager.value.emit('message_event', message)
}

// 获取历史消息
function queryHistoryMsg(isFirstLoad = false) {
  if (loadingHistory.value || !hasMoreHistory.value)
    return

  loadingHistory.value = true

  const message = { sessionId: receiveUserId.value, pageNum: historyQueryParams.pageNum, pageSize: historyQueryParams.pageSize }
  socketManager.value.emit('history_session', message)
}

// 添加滚动监听函数
function onScroll(e: any) {
  const { scrollTop } = e.detail

  // 滚动到顶部时加载更多历史记录
  if (scrollTop === 0 && hasMoreHistory.value && !loadingHistory.value) {
    console.log('滚动到顶部，加载更多历史记录')
    loadMoreHistory()
  }
}

// 加载更多历史记录
function loadMoreHistory() {
  if (loadingHistory.value || !hasMoreHistory.value)
    return

  // 记录当前第一条消息的位置
  const firstMsgIndex = 0

  // 获取当前scroll-view的高度
  uni.createSelectorQuery()
    .select('#scrollview')
    .boundingClientRect((rect: any) => {
      if (rect) {
        // 记录当前滚动位置相关数据
        currentScrollHeight = rect.height

        // 触发加载
        queryHistoryMsg()
      }
    })
    .exec()
}

// 保持滚动位置的函数
let currentScrollHeight = 0
function keepScrollPosition(newMsgCount: number) {
  setTimeout(() => {
    uni.createSelectorQuery()
      .select('#msglistview')
      .boundingClientRect((rect: any) => {
        if (rect) {
          // 计算新消息的高度
          const newContentHeight = rect.height

          // 设置新的滚动位置，使内容保持在原位置
          scrollTop.value = rpxTopx(newContentHeight - currentScrollHeight)
        }
      })
      .exec()
  }, 50)
}

// 重置分页状态
function resetPagination() {
  historyQueryParams.pageNum = 0
  loadingHistory.value = false
  hasMoreHistory.value = true
  isFirstLoad.value = true
  msgList.value = []
}

// 创建聊天会话
function createConversation() {
  createChat({ contactId: '1745881bfe664764b6ba61501c94900b' }).then((res) => {

  }).catch((error) => {
    console.log(error)
  })
}

// 页面隐藏的时候
onHide(() => {
  if (socketManager.value) {
    // socketManager.value.destroy();
  }
  // 清空缓存里的内容
  uni.removeStorage(receiveUserId.value)
})

onShow(() => {
  // 加载消息列表
  if (socketManager.value) {
    return
  }
  historyQueryParams.pageNum = 0
  msgList.value = []
  connect()
})

onLoad((options) => {
  receiveUserId.value = options.id
  asUserList.value = uni.getStorageSync(options.id)
  sessionType.value = options.sessionType
  console.log(asUserList.value)
})
</script>

<style lang="scss" scoped>
$chatContentbgc: #fd6534;
$sendBtnbgc: #4f7df5;

.pageContainer {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}

page {
  height: 100%;
  overflow: hidden;
}

.chatContainer {
  height: calc(100vh - env(safe-area-inset-top));
  position: relative;
  display: flex;
  flex-direction: column;
  overflow: hidden;

  .pb-chat-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    min-height: 0;
    overflow: hidden;

    .chat-content {
      flex: 1;
      min-height: 0;
      overflow: hidden;
      display: flex;
      flex-direction: column;

      .chat-content-box {
        flex: 1;
        min-height: 0;
        overflow-y: auto;
        -webkit-overflow-scrolling: touch;

        ::-webkit-scrollbar {
          display: none;
          width: 0 !important;
          height: 0 !important;
          -webkit-appearance: none;
          background: transparent;
          color: transparent;
        }

        .chat-body {
          display: flex;
          flex-direction: column;
          padding-bottom: 20rpx;

          .self {
            justify-content: flex-end;
          }

          .item {
            display: flex;
            padding: 23rpx 30rpx;

            .right::after {
              position: absolute;
              display: inline-block;
              content: '';
              width: 0;
              height: 0;
              left: 100%;
              top: 10px;
              border: 12rpx solid transparent;
              border-left: 12rpx solid $chatContentbgc;
            }

            .left::after {
              position: absolute;
              display: inline-block;
              content: '';
              width: 0;
              height: 0;
              top: 10px;
              right: 100%;
              border: 12rpx solid transparent;
              border-right: 12rpx solid #ffffff;
            }

            .content {
              position: relative;
              max-width: 440rpx;
              border-radius: 8rpx;
              word-wrap: break-word;
              margin: 0 24rpx;
              border-radius: 5px;
              font-size: 28rpx;
              font-family: PingFang SC;
              font-weight: 300;
              line-height: 42rpx;
              padding: 24rpx 24rpx;
            }

            .right {
              background-color: $chatContentbgc;
              color: #fff;
            }

            .left {
              background-color: #fff;
              color: #333;
            }

            .avatar {
              display: flex;
              justify-content: center;
              width: 78rpx;
              height: 78rpx;
              background: $sendBtnbgc;
              border-radius: 50rpx;
              overflow: hidden;

              image {
                align-self: center;
              }
            }
          }
        }
      }
    }

    .chat-footer {
      position: fixed;
      left: 0;
      right: 0;
      bottom: 0;
      z-index: 20;
      flex-shrink: 0;
      background: #f6f6f6;
      padding-bottom: env(safe-area-inset-bottom);
      box-sizing: border-box;
      transition: transform 0.2s ease;
      box-shadow: 0 -3px 5px 0 #e5e5e5;

      .chat-bottom {
        width: 100%;
        transition: all 0.1s ease;

        .send-msg {
          display: flex;
          align-items: flex-end;
          padding: 28rpx 30rpx;
          width: 100%;
          box-sizing: border-box;
        }

        .uni-textarea {
          flex: 1;
          min-width: 0;

          .chat-input-box {
            width: 100%;
            min-height: 75rpx;
            max-height: 500rpx;
            background: #ffffff;
            border-radius: 40rpx;
            font-size: 32rpx;
            font-family: PingFang SC;
            color: #333333;
            line-height: 1.5;
            padding: 16rpx 30rpx;
            overflow-y: auto;
            outline: none;
            word-break: break-word;
            white-space: pre-wrap;
            box-sizing: border-box;

            &:empty:before {
              content: attr(data-placeholder);
              color: #999;
            }

            &:focus:empty:before {
              content: '';
            }

            &.is-focused {
              background: #f5f5f5;
            }

            /* 表情图片样式 */
            .emoji-img {
              width: 48rpx;
              height: 48rpx;
              vertical-align: middle;
              margin: 0 4rpx;
            }
          }
        }

        .send-btn {
          display: flex;
          align-items: center;
          justify-content: center;
          width: 120rpx;
          height: 75rpx;
          border-radius: 50rpx;
          font-size: 28rpx;
          font-family: PingFang SC;
          font-weight: 500;
          line-height: 28rpx;
        }
      }
    }
  }
}

.loading-hint,
.no-more-hint {
  background: rgba(255, 255, 255, 0.8);
  margin: 10rpx 20rpx;
  border-radius: 20rpx;
  backdrop-filter: blur(10rpx);
}

.icon-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20rpx;
  border-radius: 15rpx;
  background-color: #f8f8f8;
  transition: all 0.2s;
}

.icon-item:active {
  background-color: #e0e0e0;
  transform: scale(0.95);
}

.icon-item .icon {
  font-size: 48rpx;
  margin-bottom: 10rpx;
}

.icon-label {
  font-size: 24rpx;
  color: #666;
}

.image-message {
  background-color: transparent !important;
  padding: 0 !important;
  max-width: 200px !important;
  overflow: hidden;
  border-radius: 8rpx;

  .chat-image {
    width: 100%;
    max-width: 200px;
    max-height: 300px;
    border-radius: 8rpx;
    display: block;
  }
}

::v-deep .wd-upload__preview {
  display: none;
}

::v-deep .wd-drop-menu {
  top: -8px;
}

::v-deep .wd-popup--top {
  left: 72%;
  top: 5px;
  border-radius: 8px 0 8px 8px;
  line-height: 25px;
}

::v-deep .wd-drop-menu__item-title::after {
  background: white !important;
}

::v-deep .chat-action-btn {
  flex-shrink: 0;
}
::v-deep .wd-textarea__placeholder {
  // z-index: 9;
}

.chat-bottom-anchor {
  height: 1px;
}
</style>
