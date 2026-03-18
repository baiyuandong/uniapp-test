<script lang="ts" setup>
</script>

<template>
  <view class="bg-white px-4 pt-safe">
    <view>
      aaaaaaaaaaaaaaaa
    </view>
    <wd-button @click="gotoPage('/pages/coupon/index')">
      优惠券
    </wd-button>
    <wd-button @click="gotoPage('/pages/dingdan/index')">
      订单
    </wd-button>
    <wd-button @click="gotoPage('/pages/map/index')">
      地图
    </wd-button>
    <wd-button @click="gotoPage('/pages/tts/index')">
      tts
    </wd-button>
    <wd-button @click="gotoPage('/pages/youke/index')">
      游客
    </wd-button>
    <wd-button @click="gotoPage('/pages/fapiao/index')">
      发票
    </wd-button>
    <wd-button @click="gotoPage('/pages/loading1/index')">
      loading
    </wd-button>
    <wd-button @click="gotoPage('/pages/message/index')">
      聊天
    </wd-button>
  </view>
</template>

<script setup>
import { ref } from 'vue'

defineOptions({
  name: 'Home',
})
definePage({
  // 使用 type: "home" 属性设置首页，其他页面不需要设置，默认为page
  type: 'home',
  style: {
    // 'custom' 表示开启自定义导航栏，默认 'default'
    navigationStyle: 'default',
    navigationBarTitleText: '%tabbar.home%',
  },
})

const description = ref(
  'unibest 是一个集成了多种工具和技术的 uniapp 开发模板，由 uniapp + Vue3 + Ts + Vite5 + UnoCss + VSCode 构建，模板具有代码提示、自动格式化、统一配置、代码片段等功能，并内置了许多常用的基本组件和基本功能，让你编写 uniapp 拥有 best 体验。',
)
console.log('index/index 首页打印了')

function gotoPage(url) {
  // 示例：跳转到优惠券相关页面
  // 可以根据需要修改具体的页面路径
  uni.navigateTo({
    url,
  })
}

onLoad(() => {
  console.log('测试 uni API 自动引入: onLoad')
})

const activeTab = ref(0)
const showSelectPopup = ref(false)
const showHeaderPopup = ref(false)

const selectedHeaderId = ref(null)
const currentOrder = ref(null)

const invoiceList = ref([
  {
    id: 1,
    title: '西安·敦煌7日游',
    desc: '1成人 1儿童',
    amount: 900,
    status: 'pending',
  },
])

const headerList = ref([
  {
    id: 1,
    name: '李明宇',
    type: '公司',
    email: '123456@qq.com',
  },
])

function openSelectHeader(order) {
  currentOrder.value = order
  showSelectPopup.value = true
}

function confirmInvoice() {
  showSelectPopup.value = false
  uni.showToast({ title: '已提交开票', icon: 'success' })
}

function openHeaderManage() {
  showSelectPopup.value = false
  showHeaderPopup.value = true
}

function goAddHeader() {
  uni.navigateTo({ url: '/pages/invoice/edit' })
}

function editHeader(h) {
  uni.navigateTo({
    url: `/pages/invoice/edit?id=${h.id}`,
  })
}

function deleteHeader(id) {
  headerList.value = headerList.value.filter(i => i.id !== id)
}
</script>

<style scoped>
.page {
  background: #f7f8fa;
  min-height: 100vh;
}
.list {
  padding: 12px;
}
.row {
  display: flex;
  align-items: center;
}
.between {
  justify-content: space-between;
}
.title {
  font-weight: 600;
}
.desc {
  color: #999;
  margin: 6px 0;
}
.price {
  color: #f56c6c;
  font-weight: bold;
}
.popup {
  padding: 12px;
}
.popup-title {
  font-size: 16px;
  font-weight: 600;
  margin-bottom: 12px;
}
.popup-footer {
  margin-top: 12px;
}
</style>
