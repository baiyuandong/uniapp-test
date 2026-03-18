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
/* ================================= */
/*   外层容器：128rpx × 128rpx        */
/* ================================= */
.hourglass-wrapper {
  width: 128rpx;
  height: 128rpx;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: transparent;
}

/* ================================= */
/*   沙漏主体                          */
/* ================================= */
.hourglass {
  width: 50px; /* 原始宽 */
  height: 70px; /* 原始高 */
  transform: scale(0.96); /* 约等于 49×67 rpx */
  transform-origin: center;
  animation: hourglass-rotate 2s ease-in infinite;
  transform-style: preserve-3d;
  perspective: 1000px;
  position: relative;
}

/* ================================= */
/*   删除黑点                         */
/* ================================= */
.curves:before,
.curves:after {
  display: none;
}

/* ================================= */
/*   顶部 / 底部              */
/* ================================= */
.cap-top,
.cap-bottom {
  position: absolute;
  left: 0;
  width: 50px;
  height: 5px;
  background-color: #c1c1c1;
  border-radius: 3px;
}

.cap-top {
  top: 0;
}
.cap-bottom {
  bottom: 0;
}

/* ================================= */
/*   上方玻璃                          */
/* ================================= */
.glass-top {
  position: absolute;
  top: -15px;
  left: 3px;
  width: 44px;
  height: 44px;
  background-color: #e5e5e5;
  border-radius: 50%;
  transform: rotateX(90deg);
}

/* ================================= */
/*   下方玻璃                          */
/* ================================= */
.glass {
  position: absolute;
  top: 32px;
  left: 20px;
  width: 10px;
  height: 6px;
  background-color: #e5e5e5;
  opacity: 0.6;
}
.glass:before,
.glass:after {
  content: '';
  position: absolute;
  left: -17px;
  width: 44px;
  height: 28px;
  background-color: #e5e5e5;
}
.glass:before {
  top: -26px;
  border-radius: 0 0 24px 24px;
}
.glass:after {
  bottom: -26px;
  border-radius: 24px 24px 0 0;
}

/* ================================= */
/*   沙子（纯白）                      */
/* ================================= */
.sand:before,
.sand:after {
  content: '';
  position: absolute;
  left: 6px;
  background-color: #ffffff;
}

.sand:before {
  top: 8px;
  width: 39px;
  border-radius: 3px 3px 30px 30px;
  animation: sand-fill 2s ease-in infinite;
}

.sand:after {
  border-radius: 30px 30px 3px 3px;
  animation: sand-deplete 2s ease-in infinite;
}

/* ================================= */
/*   沙流主体（白色）                  */
/* ================================= */
.sand-stream:before {
  content: '';
  position: absolute;
  left: 24px;
  width: 3px;
  background-color: #ffffff;
  animation: sand-stream1 2s ease-in infinite;
}

.sand-stream:after {
  content: '';
  position: absolute;
  top: 36px;
  left: 19px;
  border-left: 6px solid transparent;
  border-right: 6px solid transparent;
  border-bottom: 6px solid #ffffff;
  animation: sand-stream2 2s ease-in infinite;
}

/* ================================= */
/*   动画定义                          */
/* ================================= */
@keyframes hourglass-rotate {
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

@keyframes sand-stream1 {
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

@keyframes sand-stream2 {
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

@keyframes sand-fill {
  0%,
  60% {
    height: 0;
  }
  100% {
    height: 17px;
  }
}

@keyframes sand-deplete {
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
