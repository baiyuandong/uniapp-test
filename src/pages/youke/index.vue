<template>
  <view class="page">

    <!-- 顶部标题 -->
    <view class="header">导游风采</view>

    <!-- 图片列表 -->
    <view class="grid">
      <view
        v-for="(item, index) in photos"
        :key="item.id"
        class="grid-item"
        @click="toggleSelect(index)"
      >
        <image :src="item.url" mode="aspectFill" />

        <!-- 勾选图标（编辑模式） -->
        <view v-if="editing" class="check">
          <view v-if="item.checked" class="checked">✔</view>
        </view>
      </view>

      <!-- 上传占位 -->
      <view v-if="!editing" class="grid-item upload">
        <view class="upload-box">
          <view class="icon">+</view>
          <text>点击上传照片</text>
        </view>
      </view>
    </view>

    <!-- 底部按钮 -->
    <view class="footer">
      <!-- 非编辑状态 -->
      <button
        v-if="!editing"
        class="btn edit"
        @click="editing = true"
      >
        编辑照片
      </button>

      <!-- 编辑状态 -->
      <view v-else class="edit-actions">
        <button class="btn delete" @click="deleteSelected">
          删除
        </button>
        <button class="btn cancel" @click="exitEdit">
          退出编辑
        </button>
      </view>
    </view>

  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'

interface Photo {
  id: number
  url: string
  checked?: boolean
}

const editing = ref(false)

const photos = ref<Photo[]>([
  { id: 1, url: '/static/images/avatar.jpg' },
  { id: 2, url: '/static/images/avatar.jpg' },
  { id: 3, url: '/static/images/avatar.jpg' },
  { id: 4, url: '/static/images/avatar.jpg' },
  { id: 5, url: '/static/images/avatar.jpg' },
  { id: 6, url: '/static/images/avatar.jpg' },
  { id: 7, url: '/static/images/avatar.jpg' },
  { id: 8, url: '/static/images/avatar.jpg' }
])

function toggleSelect(index: number) {
  if (!editing.value) return
  photos.value[index].checked = !photos.value[index].checked
}

function deleteSelected() {
  photos.value = photos.value.filter(p => !p.checked)
}

function exitEdit() {
  editing.value = false
  photos.value.forEach(p => (p.checked = false))
}
</script>

<style scoped>
.page {
  min-height: 100vh;
  background: #f5f5f5;
}

/* 头部 */
.header {
  background: #1677ff;
  color: #fff;
  text-align: center;
  padding: 24rpx 0;
  font-size: 32rpx;
  font-weight: bold;
}

/* 网格 */
.grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 16rpx;
  padding: 16rpx;
}

.grid-item {
  position: relative;
  height: 300rpx;
  border-radius: 12rpx;
  overflow: hidden;
  background: #eee;
}

.grid-item image {
  width: 100%;
  height: 100%;
}

/* 勾选 */
.check {
  position: absolute;
  top: 12rpx;
  right: 12rpx;
}

.checked {
  width: 40rpx;
  height: 40rpx;
  background: #1677ff;
  color: #fff;
  border-radius: 50%;
  text-align: center;
  line-height: 40rpx;
  font-size: 24rpx;
}

/* 上传 */
.upload {
  background: #fff;
  border: 2rpx dashed #ccc;
}

.upload-box {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #999;
}

.upload-box .icon {
  font-size: 48rpx;
  margin-bottom: 12rpx;
}

/* 底部 */
.footer {
  padding: 24rpx;
}

.btn {
  border-radius: 12rpx;
  font-size: 28rpx;
}

.edit {
  background: #ff6a21;
  color: #fff;
}

.edit-actions {
  display: flex;
  gap: 24rpx;
}

.delete {
  flex: 1;
  background: #e60012;
  color: #fff;
}

.cancel {
  flex: 1;
  background: #999;
  color: #fff;
}
</style>
