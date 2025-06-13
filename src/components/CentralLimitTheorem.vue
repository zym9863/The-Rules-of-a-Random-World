<template>
  <div class="central-limit-theorem">
    <div class="theory-section">
      <h2>中心极限定理可视化验证</h2>
      <p class="description">
        中心极限定理表明，无论总体分布如何，当样本大小足够大时，样本均值的分布都会趋近于正态分布。
        下面的模拟器可以帮你验证这一重要定理。
      </p>
    </div>

    <div class="controls-section">
      <div class="control-group">
        <label for="population">选择总体分布：</label>
        <select id="population" v-model="selectedPopulation" class="control-input">
          <option value="uniform">均匀分布 [0,1]</option>
          <option value="exponential">指数分布 (λ=1)</option>
          <option value="bimodal">双峰分布</option>
          <option value="skewed">偏态分布</option>
        </select>
      </div>

      <div class="control-group">
        <label for="sampleSize">样本大小 (n)：</label>
        <input 
          id="sampleSize" 
          type="range" 
          min="1" 
          max="100" 
          v-model="sampleSize" 
          class="control-slider"
        >
        <span class="size-label">{{ sampleSize }}</span>
      </div>

      <div class="control-group">
        <label for="numSamples">抽样次数：</label>
        <input 
          id="numSamples" 
          type="range" 
          min="100" 
          max="10000" 
          step="100"
          v-model="numSamples" 
          class="control-slider"
        >
        <span class="size-label">{{ numSamples }}</span>
      </div>

      <div class="control-buttons">
        <button @click="runSimulation" :disabled="isRunning" class="btn btn-primary">
          开始验证
        </button>
        <button @click="resetSimulation" class="btn btn-danger">
          重置
        </button>
      </div>
    </div>

    <div class="progress-section" v-if="isRunning">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
      </div>
      <p class="progress-text">进度: {{ Math.round(progressPercentage) }}%</p>
    </div>

    <div class="results-section" v-if="sampleMeans.length > 0">
      <div class="stats-grid">
        <div class="stat-card">
          <h3>样本均值的均值</h3>
          <p class="stat-value">{{ sampleMeansAverage.toFixed(4) }}</p>
        </div>
        <div class="stat-card">
          <h3>理论期望值</h3>
          <p class="stat-value">{{ theoreticalMean.toFixed(4) }}</p>
        </div>
        <div class="stat-card">
          <h3>样本均值的标准差</h3>
          <p class="stat-value">{{ sampleMeansStd.toFixed(4) }}</p>
        </div>
        <div class="stat-card">
          <h3>理论标准误差</h3>
          <p class="stat-value">{{ theoreticalSE.toFixed(4) }}</p>
        </div>
      </div>

      <div class="normality-test" :class="{ 'normal': normalityScore > 0.95 }">
        <h3>正态性指标</h3>
        <div class="normality-score">
          <div class="score-bar">
            <div class="score-fill" :style="{ width: normalityScore * 100 + '%' }"></div>
          </div>
          <span class="score-text">{{ (normalityScore * 100).toFixed(1) }}%</span>
        </div>
        <p class="normality-description">
          {{ normalityScore > 0.95 ? '分布非常接近正态分布！' : normalityScore > 0.85 ? '分布较为接近正态分布' : '增加样本大小可能获得更好的正态性' }}
        </p>
      </div>
    </div>

    <div class="charts-section">
      <div class="chart-container">
        <h3>总体分布</h3>
        <canvas ref="populationChart" width="400" height="300"></canvas>
      </div>
      <div class="chart-container" v-if="sampleMeans.length > 0">
        <h3>样本均值分布</h3>
        <canvas ref="samplingChart" width="400" height="300"></canvas>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onUnmounted, watch } from 'vue'
import { Chart, registerables } from 'chart.js'

Chart.register(...registerables)

const selectedPopulation = ref('uniform')
const sampleSize = ref(30)
const numSamples = ref(1000)
const isRunning = ref(false)
const currentProgress = ref(0)
const sampleMeans = ref<number[]>([])
const populationChart = ref<HTMLCanvasElement>()
const samplingChart = ref<HTMLCanvasElement>()

let populationChartInstance: Chart | null = null
let samplingChartInstance: Chart | null = null

const populations = {
  uniform: {
    name: '均匀分布',
    mean: 0.5,
    variance: 1/12,
    sample: () => Math.random(),
    pdf: (x: number) => x >= 0 && x <= 1 ? 1 : 0
  },
  exponential: {
    name: '指数分布',
    mean: 1,
    variance: 1,
    sample: () => -Math.log(1 - Math.random()),
    pdf: (x: number) => x >= 0 ? Math.exp(-x) : 0
  },
  bimodal: {
    name: '双峰分布',
    mean: 0.5,
    variance: 0.29,
    sample: () => {
      // 两个正态分布的混合
      const isFirst = Math.random() < 0.5
      const mean = isFirst ? 0.2 : 0.8
      const std = 0.1
      const u1 = Math.random()
      const u2 = Math.random()
      const z0 = Math.sqrt(-2 * Math.log(u1)) * Math.cos(2 * Math.PI * u2)
      return Math.max(0, Math.min(1, mean + std * z0))
    },
    pdf: (x: number) => {
      if (x < 0 || x > 1) return 0
      // 近似双峰分布的密度
      const d1 = Math.exp(-50 * Math.pow(x - 0.2, 2))
      const d2 = Math.exp(-50 * Math.pow(x - 0.8, 2))
      return (d1 + d2) / 2
    }
  },
  skewed: {
    name: '偏态分布',
    mean: 0.33,
    variance: 0.056,
    sample: () => {
      // Beta分布 (2, 5) 的近似
      let sum = 0
      for (let i = 0; i < 7; i++) {
        sum += Math.random()
      }
      return Math.pow(sum / 7, 2)
    },
    pdf: (x: number) => x >= 0 && x <= 1 ? 6 * x * Math.pow(1 - x, 2) : 0
  }
}

const progressPercentage = computed(() => {
  return (currentProgress.value / numSamples.value) * 100
})

const currentPopulation = computed(() => {
  return populations[selectedPopulation.value as keyof typeof populations]
})

const theoreticalMean = computed(() => {
  return currentPopulation.value.mean
})

const theoreticalSE = computed(() => {
  return Math.sqrt(currentPopulation.value.variance / sampleSize.value)
})

const sampleMeansAverage = computed(() => {
  if (sampleMeans.value.length === 0) return 0
  return sampleMeans.value.reduce((sum, mean) => sum + mean, 0) / sampleMeans.value.length
})

const sampleMeansStd = computed(() => {
  if (sampleMeans.value.length === 0) return 0
  const avg = sampleMeansAverage.value
  const variance = sampleMeans.value.reduce((sum, mean) => sum + Math.pow(mean - avg, 2), 0) / sampleMeans.value.length
  return Math.sqrt(variance)
})

const normalityScore = computed(() => {
  if (sampleMeans.value.length < 100) return 0
  
  // 使用 Shapiro-Wilk 测试的简化版本
  const sortedMeans = [...sampleMeans.value].sort((a, b) => a - b)
  const n = sortedMeans.length
  const mean = sampleMeansAverage.value
  const std = sampleMeansStd.value
  
  // 计算偏度和峰度
  let skewness = 0
  let kurtosis = 0
  
  for (const value of sortedMeans) {
    const z = (value - mean) / std
    skewness += Math.pow(z, 3)
    kurtosis += Math.pow(z, 4)
  }
  
  skewness = skewness / n
  kurtosis = kurtosis / n - 3 // 超额峰度
  
  // 基于偏度和峰度计算正态性评分
  const skewnessScore = Math.max(0, 1 - Math.abs(skewness) * 2)
  const kurtosisScore = Math.max(0, 1 - Math.abs(kurtosis) * 0.5)
  
  return (skewnessScore + kurtosisScore) / 2
})

const initPopulationChart = () => {
  if (!populationChart.value) return

  const ctx = populationChart.value.getContext('2d')
  if (!ctx) return

  // 生成理论分布数据
  const points = 100
  const xValues = []
  const yValues = []
  
  for (let i = 0; i <= points; i++) {
    const x = i / points
    xValues.push(x.toFixed(2))
    yValues.push(currentPopulation.value.pdf(x))
  }

  populationChartInstance = new Chart(ctx, {
    type: 'line',
    data: {
      labels: xValues,
      datasets: [{
        label: '概率密度',
        data: yValues,
        borderColor: '#8b5cf6',
        backgroundColor: 'rgba(139, 92, 246, 0.2)',
        borderWidth: 2,
        fill: true,
        tension: 0.4
      }]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          title: {
            display: true,
            text: '值'
          }
        },
        y: {
          title: {
            display: true,
            text: '密度'
          }
        }
      },
      plugins: {
        legend: {
          display: false
        }
      }
    }
  })
}

const initSamplingChart = () => {
  if (!samplingChart.value || sampleMeans.value.length === 0) return

  const ctx = samplingChart.value.getContext('2d')
  if (!ctx) return

  // 创建直方图数据
  const bins = 30
  const min = Math.min(...sampleMeans.value)
  const max = Math.max(...sampleMeans.value)
  const binWidth = (max - min) / bins
  const histogram = new Array(bins).fill(0)
  const binCenters = []

  for (let i = 0; i < bins; i++) {
    binCenters.push(min + (i + 0.5) * binWidth)
  }

  // 计算直方图
  for (const mean of sampleMeans.value) {
    const binIndex = Math.min(Math.floor((mean - min) / binWidth), bins - 1)
    histogram[binIndex]++
  }

  // 标准化为概率密度
  const totalArea = histogram.reduce((sum, count) => sum + count, 0) * binWidth
  const density = histogram.map(count => count / totalArea)

  // 生成理论正态分布曲线
  const theoreticalX = []
  const theoreticalY = []
  const theoreticalMu = theoreticalMean.value
  const theoreticalSigma = theoreticalSE.value

  for (let i = 0; i <= 100; i++) {
    const x = min + (max - min) * i / 100
    theoreticalX.push(x.toFixed(4))
    const y = (1 / (theoreticalSigma * Math.sqrt(2 * Math.PI))) * 
              Math.exp(-0.5 * Math.pow((x - theoreticalMu) / theoreticalSigma, 2))
    theoreticalY.push(y)
  }

  if (samplingChartInstance) {
    samplingChartInstance.destroy()
  }

  samplingChartInstance = new Chart(ctx, {
    type: 'bar',
    data: {
      labels: binCenters.map(x => x.toFixed(3)),
      datasets: [
        {
          label: '样本均值分布',
          data: density,
          backgroundColor: 'rgba(59, 130, 246, 0.6)',
          borderColor: '#3b82f6',
          borderWidth: 1
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false,
      scales: {
        x: {
          title: {
            display: true,
            text: '样本均值'
          }
        },
        y: {
          title: {
            display: true,
            text: '密度'
          }
        }
      },
      plugins: {
        legend: {
          display: true
        }
      }
    }
  })

  // 添加理论正态分布曲线
  samplingChartInstance.data.datasets.push({
    type: 'line',
    label: '理论正态分布',
    data: theoreticalX.map((x, i) => ({ x: parseFloat(x), y: theoreticalY[i] })),
    borderColor: '#ef4444',
    backgroundColor: 'rgba(239, 68, 68, 0.1)',
    borderWidth: 2,
    fill: false,
    tension: 0.4,
    pointRadius: 0
  } as any)

  samplingChartInstance.update()
}

const runSimulation = async () => {
  if (isRunning.value) return

  isRunning.value = true
  currentProgress.value = 0
  sampleMeans.value = []

  const batchSize = 100
  const totalBatches = Math.ceil(numSamples.value / batchSize)

  for (let batch = 0; batch < totalBatches; batch++) {
    const currentBatchSize = Math.min(batchSize, numSamples.value - batch * batchSize)
    
    for (let i = 0; i < currentBatchSize; i++) {
      // 生成一个样本
      let sampleSum = 0
      for (let j = 0; j < sampleSize.value; j++) {
        sampleSum += currentPopulation.value.sample()
      }
      const sampleMean = sampleSum / sampleSize.value
      sampleMeans.value.push(sampleMean)
    }

    currentProgress.value = Math.min((batch + 1) * batchSize, numSamples.value)
    
    // 让UI更新
    await new Promise(resolve => setTimeout(resolve, 1))
  }

  isRunning.value = false
  initSamplingChart()
}

const resetSimulation = () => {
  isRunning.value = false
  currentProgress.value = 0
  sampleMeans.value = []
  
  if (samplingChartInstance) {
    samplingChartInstance.destroy()
    samplingChartInstance = null
  }
}

watch(selectedPopulation, () => {
  resetSimulation()
  initPopulationChart()
})

onMounted(() => {
  initPopulationChart()
})

onUnmounted(() => {
  if (populationChartInstance) {
    populationChartInstance.destroy()
  }
  if (samplingChartInstance) {
    samplingChartInstance.destroy()
  }
})
</script>

<style scoped>
.central-limit-theorem {
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

.central-limit-theorem::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  height: 4px;
  background: linear-gradient(90deg, #8b5cf6, #ec4899, #f59e0b);
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
  background: linear-gradient(135deg, #1f2937, #8b5cf6);
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
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
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
  border-color: #8b5cf6;
  box-shadow: 0 0 0 3px rgba(139, 92, 246, 0.1);
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
  background: linear-gradient(135deg, #8b5cf6, #ec4899);
  cursor: pointer;
  box-shadow: 0 2px 8px rgba(139, 92, 246, 0.3);
  transition: all 0.2s ease;
}

.control-slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 12px rgba(139, 92, 246, 0.4);
}

.control-slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  border-radius: 50%;
  background: linear-gradient(135deg, #8b5cf6, #ec4899);
  cursor: pointer;
  border: none;
  box-shadow: 0 2px 8px rgba(139, 92, 246, 0.3);
}

.size-label {
  font-size: 0.95rem;
  color: #8b5cf6;
  text-align: center;
  font-weight: 700;
  background: rgba(139, 92, 246, 0.1);
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
  background: linear-gradient(135deg, #8b5cf6, #ec4899);
  color: white;
  box-shadow: 0 4px 15px rgba(139, 92, 246, 0.3);
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px) scale(1.02);
  box-shadow: 0 8px 25px rgba(139, 92, 246, 0.4);
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

.progress-section {
  margin-bottom: 2.5rem;
  text-align: center;
  padding: 1.5rem;
  background: linear-gradient(135deg, rgba(139, 92, 246, 0.05), rgba(236, 72, 153, 0.05));
  border-radius: 16px;
  border: 1px solid rgba(139, 92, 246, 0.1);
}

.progress-bar {
  width: 100%;
  height: 12px;
  background: rgba(139, 92, 246, 0.1);
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 1rem;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #8b5cf6, #ec4899, #f59e0b);
  border-radius: 6px;
  transition: width 0.3s ease;
  box-shadow: 0 2px 8px rgba(139, 92, 246, 0.3);
}

.progress-text {
  color: #8b5cf6;
  font-weight: 700;
  font-size: 1.1rem;
}

.results-section {
  margin-bottom: 2.5rem;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 1.5rem;
  margin-bottom: 2rem;
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
  background: linear-gradient(90deg, #8b5cf6, #ec4899, #f59e0b);
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

.normality-test {
  background: linear-gradient(135deg, rgba(251, 191, 36, 0.1), rgba(253, 164, 175, 0.1));
  padding: 2rem;
  border-radius: 20px;
  text-align: center;
  transition: all 0.3s ease;
  border: 1px solid rgba(251, 191, 36, 0.2);
  box-shadow: 0 8px 25px rgba(251, 191, 36, 0.1);
}

.normality-test.normal {
  background: linear-gradient(135deg, rgba(16, 185, 129, 0.1), rgba(52, 211, 153, 0.1));
  border-color: rgba(16, 185, 129, 0.2);
  box-shadow: 0 8px 25px rgba(16, 185, 129, 0.1);
}

.normality-test h3 {
  color: #374151;
  margin-bottom: 1.5rem;
  font-weight: 700;
  font-size: 1.1rem;
}

.normality-score {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 1rem;
}

.score-bar {
  flex: 1;
  height: 12px;
  background: rgba(255, 255, 255, 0.6);
  border-radius: 6px;
  overflow: hidden;
  box-shadow: inset 0 2px 4px rgba(0, 0, 0, 0.05);
}

.score-fill {
  height: 100%;
  background: linear-gradient(90deg, #ef4444, #f59e0b, #10b981);
  border-radius: 6px;
  transition: width 0.5s ease;
  box-shadow: 0 2px 8px rgba(16, 185, 129, 0.3);
}

.score-text {
  font-weight: 800;
  color: #374151;
  font-size: 1.1rem;
  min-width: 60px;
}

.normality-description {
  color: #64748b;
  font-size: 1rem;
  font-weight: 500;
}

.charts-section {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(450px, 1fr));
  gap: 2.5rem;
}

.chart-container {
  background: linear-gradient(135deg, rgba(255, 255, 255, 0.95), rgba(248, 250, 252, 0.9));
  border-radius: 20px;
  padding: 1.5rem;
  height: 400px;
  box-shadow: 
    0 12px 35px rgba(0, 0, 0, 0.1),
    0 4px 15px rgba(0, 0, 0, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.6);
  transition: all 0.3s ease;
}

.chart-container:hover {
  transform: translateY(-2px);
  box-shadow: 
    0 16px 45px rgba(0, 0, 0, 0.12),
    0 8px 20px rgba(0, 0, 0, 0.08);
}

.chart-container h3 {
  text-align: center;
  color: #374151;
  margin-bottom: 1rem;
  font-weight: 700;
  font-size: 1.1rem;
}

.chart-container canvas {
  width: 100% !important;
  height: calc(100% - 40px) !important;
}

@media (max-width: 768px) {
  .controls-section {
    grid-template-columns: 1fr;
  }
  
  .control-buttons {
    flex-direction: column;
  }
  
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
  
  .charts-section {
    grid-template-columns: 1fr;
  }
  
  .chart-container {
    height: 300px;
  }
}
</style>
