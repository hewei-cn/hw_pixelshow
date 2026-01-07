<script setup>
import { ref, onMounted, watch, onBeforeUnmount } from 'vue'

const props = defineProps({
  image: {
    type: String,
    required: true,
  },
  pixelSize: {
    type: Number,
    default: 8,
  },
  // 动画时长（毫秒）
  duration: {
    type: Number,
    default: 1800,
  },
  // 色块开始运动的延迟间隔（毫秒），默认0表示同时开始
  delayStart: {
    type: Number,
    default: 0,
  },
  /**
   * 起始点类型：
   * - center：画布中心
   * - top-center：顶部中间
   * - bottom-center：底部中间
   */
  startPoint: {
    type: String,
    default: 'center',
    validator: (v) => ['center', 'top-center', 'bottom-center'].includes(v),
  },
  /**
   * 是否启用拖尾效果
   */
  trailEnabled: {
    type: Boolean,
    default: false,
  },
  /**
   * 拖尾淡出速度（0-1），值越大淡出越快，0表示不淡出
   */
  trailFade: {
    type: Number,
    default: 0.1,
  },
  /**
   * 起始位置 X 轴随机偏移范围（像素），默认 0 表示不偏移
   */
  startOffsetX: {
    type: Number,
    default: 0,
  },
  /**
   * 起始位置 Y 轴随机偏移范围（像素），默认 0 表示不偏移
   */
  startOffsetY: {
    type: Number,
    default: 0,
  },
  /**
   * 色块运动前是否可见
   */
  visibleBeforeStart: {
    type: Boolean,
    default: true,
  },
  /**
   * 运动曲线类型
   */
  easingType: {
    type: String,
    default: 'easeOutCubic',
    validator: (v) => ['linear', 'easeInQuad', 'easeOutQuad', 'easeInOutQuad', 'easeInCubic', 'easeOutCubic', 'easeInOutCubic', 'easeInQuart', 'easeOutQuart', 'easeInOutQuart', 'easeInBounce', 'easeOutBounce', 'easeInOutBounce', 'easeInElastic', 'easeOutElastic', 'easeInOutElastic'].includes(v),
  },
})

const canvasRef = ref(null)
let particles = []
let animationFrameId = null
let startTime = 0

function clearAnimation() {
  if (animationFrameId !== null) {
    cancelAnimationFrame(animationFrameId)
    animationFrameId = null
  }
}

// 缓动函数集合
function linear(t) {
  return t
}

function easeInQuad(t) {
  return t * t
}

function easeOutQuad(t) {
  return t * (2 - t)
}

function easeInOutQuad(t) {
  return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t
}

function easeInCubic(t) {
  return t * t * t
}

function easeOutCubic(t) {
  return 1 - Math.pow(1 - t, 3)
}

function easeInOutCubic(t) {
  return t < 0.5 ? 4 * t * t * t : (t - 1) * (2 * t - 2) * (2 * t - 2) + 1
}

function easeInQuart(t) {
  return t * t * t * t
}

function easeOutQuart(t) {
  return 1 - Math.pow(1 - t, 4)
}

function easeInOutQuart(t) {
  return t < 0.5 ? 8 * t * t * t * t : 1 - 8 * (--t) * t * t * t
}

function easeInBounce(t) {
  return 1 - easeOutBounce(1 - t)
}

function easeOutBounce(t) {
  if (t < 1 / 2.75) {
    return 7.5625 * t * t
  } else if (t < 2 / 2.75) {
    return 7.5625 * (t -= 1.5 / 2.75) * t + 0.75
  } else if (t < 2.5 / 2.75) {
    return 7.5625 * (t -= 2.25 / 2.75) * t + 0.9375
  } else {
    return 7.5625 * (t -= 2.625 / 2.75) * t + 0.984375
  }
}

function easeInOutBounce(t) {
  return t < 0.5
    ? easeInBounce(t * 2) * 0.5
    : easeOutBounce(t * 2 - 1) * 0.5 + 0.5
}

function easeInElastic(t) {
  if (t === 0) return 0
  if (t === 1) return 1
  return -Math.pow(2, 10 * (t - 1)) * Math.sin((t - 1.1) * 5 * Math.PI)
}

function easeOutElastic(t) {
  if (t === 0) return 0
  if (t === 1) return 1
  return Math.pow(2, -10 * t) * Math.sin((t - 0.1) * 5 * Math.PI) + 1
}

function easeInOutElastic(t) {
  if (t === 0) return 0
  if (t === 1) return 1
  t *= 2
  if (t < 1) {
    return -0.5 * Math.pow(2, 10 * (t - 1)) * Math.sin((t - 1.1) * 5 * Math.PI)
  }
  return 0.5 * Math.pow(2, -10 * (t - 1)) * Math.sin((t - 1.1) * 5 * Math.PI) + 1
}

// 根据 easingType 获取对应的缓动函数
function getEasingFunction(type) {
  const easingFunctions = {
    linear,
    easeInQuad,
    easeOutQuad,
    easeInOutQuad,
    easeInCubic,
    easeOutCubic,
    easeInOutCubic,
    easeInQuart,
    easeOutQuart,
    easeInOutQuart,
    easeInBounce,
    easeOutBounce,
    easeInOutBounce,
    easeInElastic,
    easeOutElastic,
    easeInOutElastic,
  }
  return easingFunctions[type] || easeOutCubic
}

function createParticlesFromImage(img, pixelSize) {
  const canvas = canvasRef.value
  if (!canvas) return

  const ctx = canvas.getContext('2d')
  const offCanvas = document.createElement('canvas')
  const offCtx = offCanvas.getContext('2d')

  const width = img.naturalWidth || img.width
  const height = img.naturalHeight || img.height

  canvas.width = width
  canvas.height = height
  offCanvas.width = width
  offCanvas.height = height

  // 将图片绘制到离屏 canvas 上
  offCtx.clearRect(0, 0, width, height)
  offCtx.drawImage(img, 0, 0, width, height)

  const imageData = offCtx.getImageData(0, 0, width, height)
  const data = imageData.data

  const centerX = width / 2
  const centerY = height / 2

  // 根据起始点类型确定粒子运动起点
  let originX = centerX
  let originY = centerY
  switch (props.startPoint) {
    case 'top-center':
      originX = centerX
      originY = 0
      break
    case 'bottom-center':
      originX = centerX
      originY = height
      break
    case 'center':
    default:
      originX = centerX
      originY = centerY
      break
  }

  const list = []

  // 按像素块采样，降低粒子数量
  for (let y = 0; y < height; y += pixelSize) {
    for (let x = 0; x < width; x += pixelSize) {
      const sampleX = Math.min(x + Math.floor(pixelSize / 2), width - 1)
      const sampleY = Math.min(y + Math.floor(pixelSize / 2), height - 1)
      const idx = (sampleY * width + sampleX) * 4
      const r = data[idx]
      const g = data[idx + 1]
      const b = data[idx + 2]
      const a = data[idx + 3]

      // 仅处理非透明像素；可以根据需要提高阈值（例如 > 30）
      if (a > 0) {
        const targetX = x
        const targetY = y

        // 为每个粒子生成随机偏移
        const { startOffsetX, startOffsetY } = props
        const randomOffsetX = startOffsetX > 0 
          ? (Math.random() - 0.5) * 2 * startOffsetX 
          : 0
        const randomOffsetY = startOffsetY > 0 
          ? (Math.random() - 0.5) * 2 * startOffsetY 
          : 0

        const actualStartX = originX + randomOffsetX
        const actualStartY = originY + randomOffsetY

        // 计算到起始点的距离，用于排序（使用实际起始位置）
        const dx = targetX - actualStartX
        const dy = targetY - actualStartY
        const distance = Math.sqrt(dx * dx + dy * dy)

        list.push({
          // 当前/起始坐标（带随机偏移）
          x: actualStartX,
          y: actualStartY,
          startX: actualStartX,
          startY: actualStartY,
          // 目标坐标：原像素块位置
          targetX,
          targetY,
          color: `rgba(${r},${g},${b},${a / 255})`,
          distance, // 保存距离用于排序
        })
      }
    }
  }

  // 按距离排序（从最近到最远）
  list.sort((a, b) => a.distance - b.distance)

  // 为每个粒子设置延迟时间
  const { delayStart } = props
  list.forEach((particle, index) => {
    particle.delay = index * delayStart // 第0个延迟0，第1个延迟delayStart，第2个延迟2*delayStart...
  })

  particles = list

  // 初始化绘制
  ctx.clearRect(0, 0, width, height)
  const { visibleBeforeStart } = props
  if (visibleBeforeStart) {
    // 如果运动前可见，绘制所有粒子在起始位置
    for (const p of particles) {
      ctx.fillStyle = p.color
      ctx.fillRect(p.startX, p.startY, pixelSize, pixelSize)
    }
  }
}

function animate(timestamp) {
  if (!startTime) startTime = timestamp

  const canvas = canvasRef.value
  if (!canvas) return
  const ctx = canvas.getContext('2d')
  const { pixelSize, duration, trailEnabled, trailFade, visibleBeforeStart, easingType } = props

  const elapsed = timestamp - startTime
  const maxDelay = particles.length > 0 ? Math.max(...particles.map(p => p.delay || 0)) : 0
  const totalDuration = duration + maxDelay

  // 检查是否所有动画都已完成
  const overallProgress = elapsed / totalDuration

  // 获取选定的缓动函数
  const easingFunction = getEasingFunction(easingType)

  // 拖尾效果：使用半透明覆盖层而不是完全清除
  if (trailEnabled && trailFade > 0) {
    // 用半透明的黑色矩形覆盖，实现淡出效果
    ctx.fillStyle = `rgba(0, 0, 0, ${trailFade})`
    ctx.fillRect(0, 0, canvas.width, canvas.height)
  } else {
    // 不使用拖尾时，完全清除画布
    ctx.clearRect(0, 0, canvas.width, canvas.height)
  }

  for (const p of particles) {
    const delay = p.delay || 0
    const particleElapsed = elapsed - delay

    // 如果延迟时间还没到
    if (particleElapsed < 0) {
      // 根据 visibleBeforeStart 决定是否绘制
      if (visibleBeforeStart) {
        ctx.fillStyle = p.color
        ctx.fillRect(p.startX, p.startY, pixelSize, pixelSize)
      }
      continue
    }

    // 计算该粒子的进度（0到1）
    const progress = Math.min(1, particleElapsed / duration)
    const eased = easingFunction(progress)

    const currentX = p.startX + (p.targetX - p.startX) * eased
    const currentY = p.startY + (p.targetY - p.startY) * eased
    ctx.fillStyle = p.color
    ctx.fillRect(currentX, currentY, pixelSize, pixelSize)
  }

  if (overallProgress < 1) {
    animationFrameId = requestAnimationFrame(animate)
  } else {
    animationFrameId = null
  }
}

function loadAndGenerate() {
  clearAnimation()
  startTime = 0
  particles = []

  if (!props.image) return

  const img = new Image()
  // 允许跨域加载静态资源（如从 public 或 CDN）
  img.crossOrigin = 'anonymous'
  img.src = props.image

  img.onload = () => {
    createParticlesFromImage(img, Math.max(1, props.pixelSize))
    animationFrameId = requestAnimationFrame(animate)
  }

  img.onerror = () => {
    // 图片加载失败时简单清空画布
    const canvas = canvasRef.value
    if (canvas) {
      const ctx = canvas.getContext('2d')
      ctx.clearRect(0, 0, canvas.width, canvas.height)
    }
  }
}

onMounted(() => {
  loadAndGenerate()
})

watch(
  () => [props.image, props.pixelSize, props.delayStart, props.startPoint, props.startOffsetX, props.startOffsetY],
  () => {
    loadAndGenerate()
  }
)

// 拖尾参数变化时不需要重新加载，只需要继续动画即可
watch(
  () => [props.trailEnabled, props.trailFade],
  () => {
    // 拖尾参数变化时，如果动画已停止，重新启动
    if (animationFrameId === null && particles.length > 0) {
      startTime = 0
      animationFrameId = requestAnimationFrame(animate)
    }
  }
)

// 缓动类型和可见性参数变化时不需要重新加载，只需要继续动画即可
watch(
  () => [props.easingType, props.visibleBeforeStart],
  () => {
    // 参数变化时，如果动画已停止，重新启动
    if (animationFrameId === null && particles.length > 0) {
      startTime = 0
      animationFrameId = requestAnimationFrame(animate)
    }
  }
)

onBeforeUnmount(() => {
  clearAnimation()
})
</script>

<template>
  <div class="pixel-fly-wrapper">
    <canvas ref="canvasRef" class="pixel-fly-canvas"></canvas>
  </div>
</template>

<style scoped>
.pixel-fly-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100%;
}

.pixel-fly-canvas {
  display: block;
  max-width: 100%;
  height: auto;
  background-color: transparent;
}
</style>


