<template>
  <div id="app">
    <header class="app-header">
      <h1>随机世界的法则</h1>
      <p class="subtitle">探索概率论的核心定理</p>
    </header>

    <nav class="navigation">
      <button 
        @click="activeTab = 'law-of-large-numbers'"
        :class="{ active: activeTab === 'law-of-large-numbers' }"
        class="nav-button"
      >
        大数定律
      </button>
      <button 
        @click="activeTab = 'central-limit-theorem'"
        :class="{ active: activeTab === 'central-limit-theorem' }"
        class="nav-button"
      >
        中心极限定理
      </button>
    </nav>

    <main class="main-content">
      <LawOfLargeNumbers v-if="activeTab === 'law-of-large-numbers'" />
      <CentralLimitTheorem v-if="activeTab === 'central-limit-theorem'" />
    </main>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import LawOfLargeNumbers from './components/LawOfLargeNumbers.vue'
import CentralLimitTheorem from './components/CentralLimitTheorem.vue'

const activeTab = ref<'law-of-large-numbers' | 'central-limit-theorem'>('law-of-large-numbers')
</script>

<style>
/* 全局重置和基础样式 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', 'SF Pro Display', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #6366f1 0%, #8b5cf6 50%, #ec4899 100%);
  background-attachment: fixed;
  min-height: 100vh;
  line-height: 1.6;
  font-feature-settings: 'cv11', 'ss01';
}

#app {
  min-height: 100vh;
  color: #1f2937;
  position: relative;
}

/* 添加装饰性背景元素 */
#app::before {
  content: '';
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: 
    radial-gradient(circle at 25% 25%, rgba(99, 102, 241, 0.1) 0%, transparent 50%),
    radial-gradient(circle at 75% 75%, rgba(236, 72, 153, 0.1) 0%, transparent 50%);
  pointer-events: none;
  z-index: -1;
}

.app-header {
  text-align: center;
  padding: 3rem 2rem 2rem;
  background: rgba(255, 255, 255, 0.12);
  backdrop-filter: blur(20px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.15);
  position: relative;
  overflow: hidden;
}

.app-header::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(45deg, transparent, rgba(255, 255, 255, 0.03), transparent);
  animation: shimmer 6s ease-in-out infinite;
}

@keyframes shimmer {
  0%, 100% { transform: translateX(-100%) translateY(-100%) rotate(45deg); }
  50% { transform: translateX(100%) translateY(100%) rotate(45deg); }
}

.app-header h1 {
  font-size: clamp(2rem, 5vw, 3rem);
  font-weight: 800;
  color: white;
  margin-bottom: 0.75rem;
  text-shadow: 0 4px 20px rgba(0, 0, 0, 0.3);
  letter-spacing: -0.02em;
  position: relative;
  z-index: 1;
}

.subtitle {
  font-size: clamp(1rem, 2.5vw, 1.25rem);
  color: rgba(255, 255, 255, 0.95);
  font-weight: 400;
  max-width: 600px;
  margin: 0 auto;
  position: relative;
  z-index: 1;
}

.navigation {
  display: flex;
  justify-content: center;
  gap: 1.5rem;
  padding: 2rem;
  background: rgba(255, 255, 255, 0.08);
  position: relative;
}

.nav-button {
  padding: 1rem 2rem;
  border: 2px solid rgba(255, 255, 255, 0.2);
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border-radius: 50px;
  cursor: pointer;
  font-weight: 600;
  font-size: 0.95rem;
  transition: all 0.4s cubic-bezier(0.4, 0, 0.2, 1);
  backdrop-filter: blur(15px);
  position: relative;
  overflow: hidden;
  min-width: 180px;
  text-align: center;
}

.nav-button::before {
  content: '';
  position: absolute;
  top: 0;
  left: -100%;
  width: 100%;
  height: 100%;
  background: linear-gradient(90deg, transparent, rgba(255, 255, 255, 0.2), transparent);
  transition: left 0.5s;
}

.nav-button:hover::before {
  left: 100%;
}

.nav-button:hover {
  background: rgba(255, 255, 255, 0.18);
  border-color: rgba(255, 255, 255, 0.4);
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.15);
}

.nav-button.active {
  background: rgba(255, 255, 255, 0.95);
  color: #4f46e5;
  border-color: rgba(255, 255, 255, 0.95);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.15);
  font-weight: 700;
}

.nav-button.active:hover {
  transform: translateY(-3px) scale(1.02);
  box-shadow: 0 12px 40px rgba(0, 0, 0, 0.2);
}

.main-content {
  padding: 2.5rem;
  max-width: 1400px;
  margin: 0 auto;
  position: relative;
}

/* 响应式设计改进 */
@media (max-width: 1024px) {
  .main-content {
    max-width: 100%;
    padding: 2rem;
  }
}

@media (max-width: 768px) {
  .app-header {
    padding: 2rem 1rem 1.5rem;
  }
  
  .navigation {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    padding: 1.5rem;
  }
  
  .nav-button {
    width: 100%;
    max-width: 280px;
    padding: 0.875rem 1.5rem;
  }
  
  .main-content {
    padding: 1.5rem;
  }
}

@media (max-width: 480px) {
  .app-header {
    padding: 1.5rem 1rem;
  }
  
  .navigation {
    padding: 1rem;
  }
  
  .main-content {
    padding: 1rem;
  }
}

/* 添加滚动条样式 */
::-webkit-scrollbar {
  width: 8px;
}

::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
}

::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* 改进焦点样式 */
.nav-button:focus {
  outline: none;
  box-shadow: 0 0 0 3px rgba(255, 255, 255, 0.3);
}

/* 添加过渡动画 */
.nav-button,
.app-header,
.navigation {
  transition: all 0.3s ease;
}
</style>
