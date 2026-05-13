<script lang="ts" setup>
import { reactive, ref } from 'vue'
import { useTokenStore } from '@/store/token'

definePage({
  style: {
    navigationBarTitleText: '登录',
  },
})

const tokenStore = useTokenStore()

// 表单数据
const loginForm = reactive({
  username: '',
  password: '',
})

// 表单验证规则
const formRules = {
  username: [
    { required: true, message: '请输入用户名' },
    { minLength: 2, message: '用户名至少2个字符' },
  ],
  password: [
    { required: true, message: '请输入密码' },
    { minLength: 6, message: '密码至少6个字符' },
  ],
}

// 验证错误信息
const formErrors = reactive({
  username: '',
  password: '',
})

// 是否显示密码
const showPassword = ref(false)

// 加载状态
const loading = ref(false)

// 验证单个字段
function validateField(field: 'username' | 'password') {
  const rules = formRules[field]
  const value = loginForm[field]
  formErrors[field] = ''

  for (const rule of rules) {
    if (rule.required && !value) {
      formErrors[field] = rule.message
      return false
    }
    if (rule.minLength && value.length < rule.minLength) {
      formErrors[field] = rule.message
      return false
    }
  }
  return true
}

// 验证整个表单
function validateForm() {
  const isUsernameValid = validateField('username')
  const isPasswordValid = validateField('password')
  return isUsernameValid && isPasswordValid
}

// 输入时清除对应错误
function onInput(field: 'username' | 'password') {
  if (formErrors[field]) {
    validateField(field)
  }
}

// 登录
async function doLogin() {
  if (!validateForm())
    return

  if (tokenStore.hasLogin) {
    uni.navigateBack()
    return
  }

  loading.value = true
  try {
    await tokenStore.login({
      username: loginForm.username,
      password: loginForm.password,
    })
    uni.navigateBack()
  }
  catch (error) {
    console.log('登录失败', error)
  }
  finally {
    loading.value = false
  }
}
</script>

<template>
  <view class="login-page">
    <!-- 顶部区域 -->
    <view class="login-header">
      <view class="login-logo">
        <text class="logo-icon">
          🔐
        </text>
      </view>
      <text class="login-title">
        欢迎回来
      </text>
      <text class="login-subtitle">
        请登录您的账号
      </text>
    </view>

    <!-- 表单区域 -->
    <view class="login-form">
      <!-- 用户名输入 -->
      <view class="form-group">
        <view class="input-wrapper" :class="{ 'input-error': formErrors.username }">
          <text class="input-icon">
            👤
          </text>
          <input
            v-model="loginForm.username"
            class="form-input"
            type="text"
            placeholder="请输入用户名"
            placeholder-class="placeholder-style"
            :maxlength="20"
            @input="onInput('username')"
            @blur="validateField('username')"
          >
        </view>
        <text v-if="formErrors.username" class="error-text">
          {{ formErrors.username }}
        </text>
      </view>

      <!-- 密码输入 -->
      <view class="form-group">
        <view class="input-wrapper" :class="{ 'input-error': formErrors.password }">
          <text class="input-icon">
            🔒
          </text>
          <input
            v-model="loginForm.password"
            class="form-input"
            :type="showPassword ? 'text' : 'password'"
            placeholder="请输入密码"
            placeholder-class="placeholder-style"
            :maxlength="30"
            @input="onInput('password')"
            @blur="validateField('password')"
          >
          <text class="toggle-pwd" @click="showPassword = !showPassword">
            {{ showPassword ? '🙈' : '👁️' }}
          </text>
        </view>
        <text v-if="formErrors.password" class="error-text">
          {{ formErrors.password }}
        </text>
      </view>

      <!-- 登录按钮 -->
      <button
        class="login-btn"
        :class="{ 'btn-loading': loading }"
        :disabled="loading"
        @click="doLogin"
      >
        {{ loading ? '登录中...' : '登 录' }}
      </button>

      <!-- 注册入口 -->
      <view class="register-link">
        <text class="register-text">
          还没有账号？
        </text>
        <text class="register-btn" @click="uni.navigateTo({ url: '/pages-auth/register' })">
          立即注册
        </text>
      </view>
    </view>

    <!-- 底部协议 -->
    <view class="login-footer">
      <text class="footer-text">
        登录即表示同意
      </text>
      <text class="footer-link">
        《用户协议》
      </text>
      <text class="footer-text">
        和
      </text>
      <text class="footer-link">
        《隐私政策》
      </text>
    </view>
  </view>
</template>

<style lang="scss" scoped>
.login-page {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background: linear-gradient(180deg, #f0f5ff 0%, #ffffff 40%);
  padding: 0 40rpx;
}

// 顶部区域
.login-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding-top: 120rpx;
  padding-bottom: 80rpx;

  .login-logo {
    width: 160rpx;
    height: 160rpx;
    background: linear-gradient(135deg, #0957de 0%, #3b82f6 100%);
    border-radius: 40rpx;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-bottom: 40rpx;
    box-shadow: 0 8rpx 32rpx rgba(9, 87, 222, 0.3);

    .logo-icon {
      font-size: 72rpx;
    }
  }

  .login-title {
    font-size: 44rpx;
    font-weight: 700;
    color: #1a1a2e;
    margin-bottom: 16rpx;
  }

  .login-subtitle {
    font-size: 28rpx;
    color: #9ca3af;
  }
}

// 表单区域
.login-form {
  flex: 1;

  .form-group {
    margin-bottom: 36rpx;

    .input-wrapper {
      display: flex;
      align-items: center;
      background: #f5f7fa;
      border-radius: 24rpx;
      padding: 0 32rpx;
      height: 100rpx;
      border: 2rpx solid transparent;
      transition: all 0.2s;

      &:focus-within {
        background: #ffffff;
        border-color: #0957de;
        box-shadow: 0 0 0 4rpx rgba(9, 87, 222, 0.1);
      }

      &.input-error {
        border-color: #ef4444;
        background: #ffffff;

        &:focus-within {
          box-shadow: 0 0 0 4rpx rgba(239, 68, 68, 0.1);
        }
      }

      .input-icon {
        font-size: 36rpx;
        margin-right: 20rpx;
        flex-shrink: 0;
      }

      .form-input {
        flex: 1;
        height: 100%;
        font-size: 30rpx;
        color: #1a1a2e;
        background: transparent;
      }

      .toggle-pwd {
        font-size: 32rpx;
        padding: 10rpx;
        flex-shrink: 0;
      }
    }

    .error-text {
      display: block;
      font-size: 24rpx;
      color: #ef4444;
      margin-top: 10rpx;
      padding-left: 32rpx;
    }
  }

  .login-btn {
    width: 100%;
    height: 100rpx;
    background: linear-gradient(135deg, #0957de 0%, #3b82f6 100%);
    border-radius: 50rpx;
    color: #ffffff;
    font-size: 34rpx;
    font-weight: 600;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-top: 20rpx;
    border: none;
    box-shadow: 0 8rpx 32rpx rgba(9, 87, 222, 0.35);
    transition: all 0.2s;

    &:active {
      transform: scale(0.97);
      box-shadow: 0 4rpx 16rpx rgba(9, 87, 222, 0.25);
    }

    &.btn-loading {
      opacity: 0.7;
    }

    &::after {
      border: none;
    }
  }

  .register-link {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 40rpx;
    gap: 8rpx;

    .register-text {
      font-size: 28rpx;
      color: #9ca3af;
    }

    .register-btn {
      font-size: 28rpx;
      color: #0957de;
      font-weight: 500;
    }
  }
}

// 底部协议
.login-footer {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  padding: 40rpx 0;
  gap: 4rpx;

  .footer-text {
    font-size: 24rpx;
    color: #9ca3af;
  }

  .footer-link {
    font-size: 24rpx;
    color: #0957de;
  }
}

// 统一重置按钮边框
button {
  padding: 0;
  line-height: 100rpx;
}

// placeholder 样式
.placeholder-style {
  color: #c0c4cc;
  font-size: 30rpx;
}
</style>
