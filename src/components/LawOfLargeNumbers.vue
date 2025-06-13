<template>
  <div class="law-of-large-numbers">
    <div class="theory-section">
      <h2>大数定律交互式模拟器</h2>
      <p class="description">
        大数定律告诉我们，当试验次数足够多时，随机变量的样本均值会趋近于其理论期望值。
        通过下面的模拟器，你可以直观地观察这一现象。
      </p>
    </div>

    <div class="controls-section">
      <div class="control-group">
        <label for="distribution">选择概率分布：</label>
        <select id="distribution" v-model="selectedDistribution" class="control-input">
          <option value="coin">抛硬币 (期望值: 0.5)</option>
          <option value="dice">掷骰子 (期望值: 3.5)</option>
          <option value="normal">正态分布 (μ=50, σ=10)</option>
          <option value="exponential">指数分布 (λ=0.5, 期望值: 2)</option>
        </select>
      </div>

      <div class="control-group">
        <label for="speed">模拟速度：</label>
        <input 
          id="speed" 
          type="range" 
          min="1" 
          max="100" 
          v-model="simulationSpeed" 
          class="control-slider"
        >
        <span class="speed-label">{{ simulationSpeed }}次/秒</span>
      </div>

      <div class="control-buttons">
        <button @click="startSimulation" :disabled="isRunning" class="btn btn-primary">
          开始模拟
        </button>
        <button @click="stopSimulation" :disabled="!isRunning" class="btn btn-secondary">
          停止模拟
        </button>
        <button @click="resetSimulation" class="btn btn-danger">
          重置
        </button>
      </div>
    </div>

    <div class="stats-section">
      <div class="stat-card">
        <h3>试验次数</h3>
        <p class="stat-value">{{ trials }}</p>
      </div>
      <div class="stat-card">
        <h3>当前样本均值</h3>
        <p class="stat-value">{{ currentMean.toFixed(4) }}</p>
      </div>
      <div class="stat-card">
        <h3>理论期望值</h3>
        <p class="stat-value">{{ theoreticalMean.toFixed(4) }}</p>
      </div>
      <div class="stat-card">
        <h3>偏差</h3>
        <p class="stat-value" :class="{ 'converging': Math.abs(currentMean - theoreticalMean) < 0.1 }">
          {{ Math.abs(currentMean - theoreticalMean).toFixed(4) }}
        </p>
      </div>
    </div>

    <div class="chart-section">
      <canvas ref="chartCanvas" width="800" height="400"></canvas>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const selectedDistribution = ref('coin')
const simulationSpeed = ref(10)
const isRunning = ref(false)
const trials = ref(0)
const sampleSum = ref(0)
const chartCanvas = ref<HTMLCanvasElement>()
let chart: Chart | null = null
let animationId: number | null = null

const distributions = {
  coin: {
    name: '抛硬币',
    mean: 0.5,
    sample: () => Math.random() < 0.5 ? 0 : 1
  },
  dice: {
    name: '掷骰子',
    mean: 3.5,
    sample: () => Math.floor(Math.random() * 6) + 1
  },
  normal: {
    name: '正态分布',
    mean: 50,
    sample: () => {
      // Box-Muller变换生成正态分布
      const u1 = Math.random()
      const u2 = Math.random()
      const z0 = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2)
      return 50 + 10 * z0
    }
  },
  exponential: {
    name: '指数分布',
    mean: 2,
    sample: () => -2 * Math.log(1 - Math.random())
  }
}

const currentMean = computed(() => {
  return trials.value > 0 ? sampleSum.value / trials.value : 0
})

const theoreticalMean = computed(() => {
  return distributions[selectedDistribution.value as keyof typeof distributions].mean
})

const initChart = () => {
  if (!chartCanvas.value) return

  const ctx = chartCanvas.value.getContext('2d')
  if (!ctx) return

  chart = new Chart(ctx, {
    type: 'line',
    data: {
      labels: [],
      datasets: [
        {
          label: '样本均值',
          data: [],
          borderColor: '#3b82f6',
          backgroundColor: 'rgba(59, 130, 246, 0.1)',
          borderWidth: 2,
          fill: false,
          tension: 0.1
        },
        {
          label: '理论期望值',
          data: [],
          borderColor: '#ef4444',
          backgroundColor: 'rgba(239, 68, 68, 0.1)',
          borderWidth: 2,
          borderDash: [5, 5],
          fill: false,
          pointRadius: 0
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      animation: {
        duration: 0
      },
      scales: {
        x: {
          title: {
            display: true,
            text: '试验次数'
          },
          grid: {
            color: 'rgba(255, 255, 255, 0.1)'
          },
          ticks: {
            color: 'rgba(255, 255, 255, 0.8)'
          }
        },
        y: {
          title: {
            display: true,
            text: '样本均值'
          },
          grid: {
            color: 'rgba(255, 255, 255, 0.1)'
          },
          ticks: {
            color: 'rgba(255, 255, 255, 0.8)'
          }
        }
      },
      plugins: {
        legend: {
          labels: {
            color: 'rgba(255, 255, 255, 0.9)'
          }
        }
      }
    }
  })
}

const updateChart = () => {
  if (!chart) return

  const maxPoints = 1000
  const currentTrial = trials.value
  const mean = currentMean.value
  const theoreticalValue = theoreticalMean.value

  // 添加新数据点
  if (chart.data.labels!.length >= maxPoints) {
    chart.data.labels!.shift()
    chart.data.datasets[0].data.shift()
    chart.data.datasets[1].data.shift()
  }

  chart.data.labels!.push(currentTrial.toString())
  chart.data.datasets[0].data.push(mean)
  chart.data.datasets[1].data.push(theoreticalValue)

  chart.update('none')
}

const generateSample = () => {
  const distribution = distributions[selectedDistribution.value as keyof typeof distributions]
  return distribution.sample()
}

const runSimulation = () => {
  const sample = generateSample()
  sampleSum.value += sample
  trials.value += 1
  updateChart()
}

const startSimulation = () => {
  if (isRunning.value) return
  
  isRunning.value = true
  const interval = 1000 / simulationSpeed.value

  const simulate = () => {
    if (!isRunning.value) return

    runSimulation()
    
    setTimeout(() => {
      if (isRunning.value) {
        animationId = requestAnimationFrame(simulate)
      }
    }, interval)
  }

  simulate()
}

const stopSimulation = () => {
  isRunning.value = false
  if (animationId) {
    cancelAnimationFrame(animationId)
    animationId = null
  }
}

const resetSimulation = () => {
  stopSimulation()
  trials.value = 0
  sampleSum.value = 0
  
  if (chart) {
    chart.data.labels = []
    chart.data.datasets[0].data = []
    chart.data.datasets[1].data = []
    chart.update()
  }
}

onMounted(() => {
  initChart()
})

onUnmounted(() => {
  stopSimulation()
  if (chart) {
    chart.destroy()
  }
})
</script>

<style scoped>
.law-of-large-numbers {
  background: rgba(255, 255, 255, 0.98);
  border-radius: 24px;
  padding: 2.5rem;
  box-shadow: 
    0 20px 60px rgba(0, 0, 0, 0.15),
    0 8px 25px rgba(0, 0, 0, 0.1),
    inset 0 1px 0 rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.3);
  position: relative;
  overflow: hidden;
}

.law-of-large-numbers::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #6366f1, #8b5cf6, #ec4899);
  border-radius: 24px 24px 0 0;
}

.theory-section {
  margin-bottom: 2.5rem;
  text-align: center;
  position: relative;
  z-index: 1;
}

.theory-section h2 {
  color: #1f2937;
  margin-bottom: 1.25rem;
  font-size: clamp(1.5rem, 4vw, 2rem);
  font-weight: 700;
  letter-spacing: -0.02em;
  background: linear-gradient(135deg, #1f2937, #4f46e5);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.description {
  color: #64748b;
  line-height: 1.7;
  max-width: 850px;
  margin: 0 auto;
  font-size: 1.05rem;
  font-weight: 400;
}

.controls-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 2rem;
  margin-bottom: 2.5rem;
  padding: 2rem;
  background: linear-gradient(135deg, rgba(249, 250, 251, 0.9), rgba(243, 244, 246, 0.8));
  border-radius: 20px;
  border: 1px solid rgba(255, 255, 255, 0.5);
  box-shadow: 0 8px 25px rgba(0, 0, 0, 0.08);
}

.control-group {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.control-group label {
  font-weight: 700;
  color: #374151;
  font-size: 0.95rem;
  letter-spacing: 0.01em;
}

.control-input {
  padding: 0.875rem 1rem;
  border: 2px solid #e5e7eb;
  border-radius: 12px;
  font-size: 1rem;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  background: rgba(255, 255, 255, 0.8);
  font-weight: 500;
}

.control-input:focus {
  outline: none;
  border-color: #6366f1;
  box-shadow: 0 0 0 3px rgba(99, 102, 241, 0.1);
  background: rgba(255, 255, 255, 1);
}

.control-input:hover {
  border-color: #d1d5db;
}

.control-slider {
  width: 100%;
  height: 8px;
  border-radius: 4px;
  background: #e5e7eb;
  outline: none;
  -webkit-appearance: none;
  appearance: none;
}

.control-slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  appearance: none;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.3);
  transition: all 0.2s ease;
}

.control-slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(99, 102, 241, 0.4);
}

.control-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 8px rgba(99, 102, 241, 0.3);
}

.speed-label {
  font-size: 0.95rem;
  color: #6366f1;
  text-align: center;
  font-weight: 700;
  background: rgba(99, 102, 241, 0.1);
  padding: 0.25rem 0.75rem;
  border-radius: 20px;
  margin-top: 0.25rem;
}

.control-buttons {
  display: flex;
  gap: 0.75rem;
  align-items: end;
}

.btn {
  padding: 0.875rem 1.25rem;
  border: none;
  border-radius: 12px;
  font-weight: 700;
  font-size: 0.95rem;
  cursor: pointer;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  flex: 1;
  position: relative;
  overflow: hidden;
}

.btn::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.btn:hover::before {
  left: 100%;
}

.btn-primary {
  background: linear-gradient(135deg, #6366f1, #8b5cf6);
  color: white;
  box-shadow: 0 4px 15px rgba(99, 102, 241, 0.3);
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(99, 102, 241, 0.4);
}

.btn-secondary {
  background: linear-gradient(135deg, #64748b, #475569);
  color: white;
  box-shadow: 0 4px 15px rgba(100, 116, 139, 0.3);
}

.btn-secondary:hover:not(:disabled) {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(100, 116, 139, 0.4);
}

.btn-danger {
  background: linear-gradient(135deg, #ef4444, #dc2626);
  color: white;
  box-shadow: 0 4px 15px rgba(239, 68, 68, 0.3);
}

.btn-danger:hover {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(239, 68, 68, 0.4);
}

.btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none !important;
  box-shadow: none !important;
}

.stats-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2.5rem;
}

.stat-card {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.9), rgba(248, 250, 252, 0.8));
  padding: 2rem 1.5rem;
  border-radius: 16px;
  text-align: center;
  box-shadow: 
    0 8px 25px rgba(0, 0, 0, 0.08),
    0 3px 10px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.6);
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  position: relative;
  overflow: hidden;
}

.stat-card::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 3px;
  background: linear-gradient(90deg, #6366f1, #8b5cf6, #ec4899);
}

.stat-card:hover {
  transform: translateY(-4px) scale(1.02);
  box-shadow: 
    0 15px 40px rgba(0, 0, 0, 0.12),
    0 8px 20px rgba(0, 0, 0, 0.08);
}

.stat-card h3 {
  color: #64748b;
  font-size: 0.85rem;
  margin-bottom: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.stat-value {
  font-size: 1.75rem;
  font-weight: 800;
  color: #1e293b;
  letter-spacing: -0.02em;
}

.stat-value.converging {
  color: #059669;
  animation: pulse-green 2s ease-in-out infinite;
}

@keyframes pulse-green {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.8; }
}

.chart-section {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95), rgba(248, 250, 252, 0.9));
  border-radius: 20px;
  padding: 1.5rem;
  height: 450px;
  position: relative;
  box-shadow: 
    0 12px 35px rgba(0, 0, 0, 0.1),
    0 4px 15px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.6);
}

.chart-section canvas {
  width: 100% !important;
  height: 100% !important;
}

@media (max-width: 768px) {
  .controls-section {
    grid-template-columns: 1fr;
  }
  
  .control-buttons {
    flex-direction: column;
  }
  
  .stats-section {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .chart-section {
    height: 300px;
  }
}
</style>
