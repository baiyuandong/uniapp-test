<template>
  <!-- From Uiverse.io by SouravBandyopadhyay -->
  <view class="hourglass-wrapper">
    <view class="hourglassBackground">
      <view class="hourglassContainer">
        <view class="hourglassCurves" />
        <view class="hourglassCapTop" />
        <view class="hourglassGlassTop" />
        <view class="hourglassSand" />
        <view class="hourglassSandStream" />
        <view class="hourglassCapBottom" />
        <view class="hourglassGlass" />
      </view>
    </view>
  </view>
</template>

<script lang="ts" setup>
</script>

<style lang="scss" scoped>
/* 外层：128rpx */
.hourglass-wrapper {
  width: 128rpx;
  height: 128rpx;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* 背景圆 */
.hourglassBackground {
  position: relative;
  width: 130px;
  height: 130px;
  border-radius: 50%;
  background-color: #f7f7f7; /* 接近你图里的浅灰白 */
  transform: scale(0.985);
  transform-origin: center;
}

/* 沙漏主体 */
.hourglassContainer {
  position: absolute;
  top: 30px;
  left: 40px;
  width: 50px;
  height: 70px;
  transform: scale(0.96);
  transform-origin: center;

  animation: hourglassRotate 2s ease-in infinite;
  transform-style: preserve-3d;
  perspective: 1000px;
}

/* ==================== 顶部 / 底部 ==================== */

.hourglassCapTop,
.hourglassCapBottom {
  position: absolute;
  left: 0;
  width: 50px;
  height: 5px;
  background-color: #dcdcdc;
  border-radius: 3px;
}

.hourglassCapTop {
  top: 0;
}

.hourglassCapBottom {
  bottom: 0;
}

/* ==================== 玻璃 ==================== */

.hourglassGlassTop {
  transform: rotateX(90deg);
  position: absolute;
  top: -16px;
  left: 3px;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background-color: #e5e5e5;
}

.hourglassGlass {
  position: absolute;
  top: 32px;
  left: 20px;
  width: 10px;
  height: 6px;
  background-color: #e5e5e5;
  opacity: 0.6;
}

.hourglassGlass:before,
.hourglassGlass:after {
  content: '';
  position: absolute;
  left: -17px;
  width: 44px;
  height: 28px;
  background-color: #ededed;
}

.hourglassGlass:before {
  top: -27px;
  border-radius: 0 0 24px 24px;
}

.hourglassGlass:after {
  bottom: -27px;
  border-radius: 24px 24px 0 0;
}

/* ==================== ❌ 去掉中间两个黑点 ==================== */

.hourglassCurves:before,
.hourglassCurves:after {
  display: none;
}

/* ==================== 沙子（纯白） ==================== */

.hourglassSand:before,
.hourglassSand:after {
  content: '';
  position: absolute;
  left: 6px;
  background-color: #ffffff; /* 白沙 */
}

.hourglassSand:before {
  top: 8px;
  width: 39px;
  border-radius: 3px 3px 30px 30px;
  animation: sandFillup 2s ease-in infinite;
}

.hourglassSand:after {
  border-radius: 30px 30px 3px 3px;
  animation: sandDeplete 2s ease-in infinite;
}

/* 沙流 */
.hourglassSandStream:before {
  content: '';
  position: absolute;
  left: 24px;
  width: 3px;
  background-color: #ffffff;
  animation: sandStream1 2s ease-in infinite;
}

.hourglassSandStream:after {
  content: '';
  position: absolute;
  top: 36px;
  left: 19px;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-bottom: 6px solid #ffffff;
  animation: sandStream2 2s ease-in infinite;
}

/* ==================== 动画（原样保留） ==================== */

@keyframes hourglassRotate {
  0% {
    transform: rotateX(0deg) scale(0.96);
  }
  50% {
    transform: rotateX(180deg) scale(0.96);
  }
  100% {
    transform: rotateX(180deg) scale(0.96);
  }
}

@keyframes sandStream1 {
  0% {
    height: 0;
    top: 35px;
  }
  60% {
    height: 35px;
    top: 8px;
  }
  100% {
    height: 0;
    top: 8px;
  }
}

@keyframes sandStream2 {
  0%,
  50%,
  100% {
    opacity: 0;
  }
  51%,
  90% {
    opacity: 1;
  }
}

@keyframes sandFillup {
  0%,
  60% {
    height: 0;
  }
  100% {
    height: 17px;
  }
}

@keyframes sandDeplete {
  0% {
    top: 45px;
    height: 17px;
    width: 38px;
    left: 6px;
  }
  90% {
    top: 41px;
    height: 0;
    width: 10px;
    left: 20px;
  }
}
</style>
