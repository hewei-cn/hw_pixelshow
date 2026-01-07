<script setup>
import { ref } from 'vue'
import PixelFlyCanvas from './components/PixelFlyCanvas.vue'

const imageUrl = ref('/img.png')
const pixelSize = ref(8)
const duration = ref(1800)
const delayStart = ref(0)
const startPoint = ref('center')
const startOffsetX = ref(0)
const startOffsetY = ref(0)
const visibleBeforeStart = ref(true)
const easingType = ref('easeOutCubic')
const trailEnabled = ref(false)
const trailFade = ref(0.1)
</script>

<template>
  <main class="app">
    <h1 class="title">Pixel Fly Canvas Demo</h1>
    <p class="desc">
      下方组件会将图片的非透明像素转为可飞行的像素块，从中心飞向原始位置。
      你可以通过右侧面板，实时调节所有参数并观察效果。
    </p>
    <section class="panel">
      <div class="controls">
        <div class="control-group">
          <label>
            图片地址 (image)
          </label>
          <input
            v-model="imageUrl"
            type="text"
            class="text-input"
            placeholder="/img.png 或其他图片 URL"
          />
        </div>

        <div class="control-group">
          <label>
            像素块大小 (pixelSize)
            <span class="value">{{ pixelSize }}</span>
          </label>
          <input
            v-model.number="pixelSize"
            type="range"
            min="2"
            max="30"
            step="1"
          />
        </div>

        <div class="control-group">
          <label>
            动画时长 (duration, ms)
            <span class="value">{{ duration }}</span>
          </label>
          <input
            v-model.number="duration"
            type="range"
            min="500"
            max="4000"
            step="100"
          />
        </div>

        <div class="control-group">
          <label>
            延迟间隔 (delayStart, ms)
            <span class="value">{{ delayStart }}</span>
          </label>
          <input
            v-model.number="delayStart"
            type="range"
            min="0"
            max="50"
            step="1"
          />
        </div>

        <div class="control-group">
          <label>起始点类型 (startPoint)</label>
          <select v-model="startPoint">
            <option value="center">center（中心）</option>
            <option value="top-center">top-center（顶部中心）</option>
            <option value="bottom-center">bottom-center（底部中心）</option>
          </select>
        </div>

        <div class="control-group">
          <label>
            起始位置 X 轴偏移 (startOffsetX, px)
            <span class="value">{{ startOffsetX }}</span>
          </label>
          <input
            v-model.number="startOffsetX"
            type="range"
            min="0"
            max="200"
            step="5"
          />
        </div>

        <div class="control-group">
          <label>
            起始位置 Y 轴偏移 (startOffsetY, px)
            <span class="value">{{ startOffsetY }}</span>
          </label>
          <input
            v-model.number="startOffsetY"
            type="range"
            min="0"
            max="200"
            step="5"
          />
        </div>

        <div class="control-group">
          <label>
            <input
              v-model="visibleBeforeStart"
              type="checkbox"
              class="checkbox-input"
            />
            运动前可见 (visibleBeforeStart)
          </label>
        </div>

        <div class="control-group">
          <label>运动曲线类型 (easingType)</label>
          <select v-model="easingType">
            <option value="linear">linear（线性）</option>
            <option value="easeInQuad">easeInQuad（二次缓入）</option>
            <option value="easeOutQuad">easeOutQuad（二次缓出）</option>
            <option value="easeInOutQuad">easeInOutQuad（二次缓入缓出）</option>
            <option value="easeInCubic">easeInCubic（三次缓入）</option>
            <option value="easeOutCubic">easeOutCubic（三次缓出）</option>
            <option value="easeInOutCubic">easeInOutCubic（三次缓入缓出）</option>
            <option value="easeInQuart">easeInQuart（四次缓入）</option>
            <option value="easeOutQuart">easeOutQuart（四次缓出）</option>
            <option value="easeInOutQuart">easeInOutQuart（四次缓入缓出）</option>
            <option value="easeInBounce">easeInBounce（弹跳缓入）</option>
            <option value="easeOutBounce">easeOutBounce（弹跳缓出）</option>
            <option value="easeInOutBounce">easeInOutBounce（弹跳缓入缓出）</option>
            <option value="easeInElastic">easeInElastic（弹性缓入）</option>
            <option value="easeOutElastic">easeOutElastic（弹性缓出）</option>
            <option value="easeInOutElastic">easeInOutElastic（弹性缓入缓出）</option>
          </select>
        </div>

        <div class="control-group">
          <label>
            <input
              v-model="trailEnabled"
              type="checkbox"
              class="checkbox-input"
            />
            启用拖尾效果 (trailEnabled)
          </label>
        </div>

        <div v-if="trailEnabled" class="control-group">
          <label>
            拖尾淡出速度 (trailFade)
            <span class="value">{{ trailFade.toFixed(2) }}</span>
          </label>
          <input
            v-model.number="trailFade"
            type="range"
            min="0.01"
            max="0.5"
            step="0.01"
          />
        </div>
      </div>

      <!-- 使用 public/img.png，你也可以传入其他图片 URL -->
      <div class="canvas-container">
        <PixelFlyCanvas
          :image="imageUrl"
          :pixel-size="pixelSize"
          :duration="duration"
          :delay-start="delayStart"
          :start-point="startPoint"
          :start-offset-x="startOffsetX"
          :start-offset-y="startOffsetY"
          :visible-before-start="visibleBeforeStart"
          :easing-type="easingType"
          :trail-enabled="trailEnabled"
          :trail-fade="trailFade"
        />
      </div>
    </section>
  </main>
</template>

<style scoped>
.app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: flex-start;
  padding: 40px 16px;
  box-sizing: border-box;
  background: radial-gradient(circle at top, #1f2933 0, #020617 60%);
  color: #f9fafb;
}

.title {
  font-size: 28px;
  margin-bottom: 8px;
}

.desc {
  margin-bottom: 16px;
  font-size: 14px;
  color: #e5e7eb;
}

.panel {
  width: min(720px, 100%);
  display: grid;
  grid-template-columns: minmax(0, 1.2fr) minmax(0, 1.5fr);
  gap: 16px;
}

.controls {
  padding: 16px;
  border-radius: 16px;
  background: rgba(15, 23, 42, 0.85);
  box-shadow: 0 18px 40px rgba(15, 23, 42, 0.9);
  backdrop-filter: blur(18px);
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
  margin-bottom: 12px;
  font-size: 13px;
}

.control-group label {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #e5e7eb;
}

.control-group .value {
  font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
  font-size: 12px;
  color: #a5b4fc;
}

.control-group input[type='range'] {
  width: 100%;
}

.control-group .text-input {
  width: 100%;
  padding: 6px 8px;
  border-radius: 6px;
  border: 1px solid rgba(148, 163, 184, 0.6);
  background-color: rgba(15, 23, 42, 0.95);
  color: #e5e7eb;
  font-size: 13px;
}

.control-group select {
  padding: 4px 8px;
  border-radius: 6px;
  border: 1px solid rgba(148, 163, 184, 0.6);
  background-color: rgba(15, 23, 42, 0.95);
  color: #e5e7eb;
}

.control-group .checkbox-input {
  margin-right: 8px;
  width: 16px;
  height: 16px;
  cursor: pointer;
}

.canvas-container {
  width: min(480px, 100%);
  border-radius: 16px;
  padding: 16px;
  background: rgba(15, 23, 42, 0.85);
  box-shadow: 0 24px 60px rgba(15, 23, 42, 0.9);
  backdrop-filter: blur(18px);
}

@media (max-width: 720px) {
  .panel {
    grid-template-columns: minmax(0, 1fr);
  }
}
</style>

