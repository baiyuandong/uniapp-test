<template>
  <view class="container">
    <view class="title">文字转语音（TTS 测试）</view>

    <textarea
      v-model="text"
      class="textarea"
      placeholder="请输入要转换成语音的文字"
    />

    <button
      class="btn"
      :loading="loading"
      @click="handleTTS"
    >
      {{ loading ? '生成中...' : '生成并播放' }}
    </button>

    <view v-if="audioUrl" class="audio-box">
      <text class="label">生成成功，点击播放：</text>
      <audio
        :src="audioUrl"
        controls
      />
    </view>
  </view>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const text = ref('')
const audioUrl = ref('')
const loading = ref(false)

const TTS_API = 'http://192.168.31.131:3000/tts'

async function handleTTS() {
  if (!text.value.trim()) {
    uni.showToast({ title: '请输入文字', icon: 'none' })
    return
  }

  loading.value = true
  audioUrl.value = ''

  try {
    const res = await uni.request({
      url: TTS_API,
      method: 'POST',
      header: {
        'Content-Type': 'application/json'
      },
      data: {
        text: text.value
      }
    })

    if (res.statusCode !== 200) {
      throw new Error('请求失败')
    }

    audioUrl.value = res.data.url
  } catch (err) {
    console.error(err)
    uni.showToast({
      title: '生成失败',
      icon: 'none'
    })
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.container {
  padding: 32rpx;
}

.title {
  font-size: 36rpx;
  font-weight: bold;
  margin-bottom: 24rpx;
}

.textarea {
  width: 100%;
  min-height: 200rpx;
  border: 1px solid #ddd;
  padding: 16rpx;
  box-sizing: border-box;
  border-radius: 8rpx;
  margin-bottom: 24rpx;
}

.btn {
  background-color: #1677ff;
  color: #fff;
  border-radius: 8rpx;
}

.audio-box {
  margin-top: 32rpx;
}

.label {
  display: block;
  margin-bottom: 12rpx;
  color: #333;
}
</style>
