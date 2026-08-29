<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  heartRate: Number,
  isConnected: Boolean
})

const emit = defineEmits(['toggle-history'])

const heartRateHistory = ref([])
const maxHistory = 60 // Keep last 60 seconds of data

watch(() => props.heartRate, (newRate) => {
  if (newRate > 0 && props.isConnected) {
    const now = new Date()
    heartRateHistory.value.push({
      rate: newRate,
      time: now.toLocaleTimeString('zh-CN', { hour12: false })
    })
    
    // Keep only the last maxHistory entries
    if (heartRateHistory.value.length > maxHistory) {
      heartRateHistory.value.shift()
    }
  }
})

const getHeartRateZone = (rate) => {
  if (rate < 50) return { name: '过缓', color: '#95a5a6', description: '心率过低，请休息' }
  if (rate < 100) return { name: '正常', color: '#4ecdc4', description: '静息心率范围' }
  if (rate < 130) return { name: '燃脂', color: '#f39c12', description: '最佳燃脂区间' }
  if (rate < 160) return { name: '有氧', color: '#e74c3c', description: '心肺强化区间' }
  if (rate < 180) return { name: '无氧', color: '#9b59b6', description: '极限训练区间' }
  return { name: '危险', color: '#c0392b', description: '请立即停止运动' }
}

const currentZone = computed(() => {
  return getHeartRateZone(props.heartRate)
})

const averageHeartRate = computed(() => {
  if (heartRateHistory.value.length === 0) return 0
  const sum = heartRateHistory.value.reduce((acc, item) => acc + item.rate, 0)
  return Math.round(sum / heartRateHistory.value.length)
})

const maxHeartRate = computed(() => {
  if (heartRateHistory.value.length === 0) return 0
  return Math.max(...heartRateHistory.value.map(item => item.rate))
})

const minHeartRate = computed(() => {
  if (heartRateHistory.value.length === 0) return 0
  return Math.min(...heartRateHistory.value.map(item => item.rate))
})
</script>

<template>
  <div class="heart-rate-monitor">
    <!-- Main Display -->
    <div class="monitor-display">
      <div class="display-header">
        <h2>实时心率</h2>
        <button @click="emit('toggle-history')" class="btn-history">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <path d="M13 3c-4.97 0-9 4.03-9 9H1l3.89 3.89.07.14L9 12H6c0-3.87 3.13-7 7-7s7 3.13 7 7-3.13 7-7 7c-1.93 0-3.68-.79-4.94-2.06l-1.42 1.42C8.27 19.99 10.51 21 13 21c4.97 0 9-4.03 9-9s-4.03-9-9-9zm-1 5v5l4.28 2.54.72-1.21-3.5-2.08V8H12z"/>
          </svg>
          历史记录
        </button>
      </div>

      <div class="heart-display">
        <div class="heart-wrapper">
          <svg class="heart-bg" viewBox="0 0 200 200">
            <circle cx="100" cy="100" r="90" fill="none" stroke="rgba(255,255,255,0.1)" stroke-width="2"/>
            <circle 
              cx="100" 
              cy="100" 
              r="90" 
              fill="none" 
              :stroke="currentZone.color" 
              stroke-width="8"
              stroke-dasharray="565"
              :stroke-dashoffset="565 - (565 * (props.heartRate / 200))"
              stroke-linecap="round"
              transform="rotate(-90 100 100)"
              class="progress-ring"
            />
          </svg>
          
          <div class="heart-content">
            <svg v-if="isConnected" class="heart-icon" viewBox="0 0 24 24" fill="currentColor">
              <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
            </svg>
            
            <div v-else class="disconnected-icon">
              <svg viewBox="0 0 24 24" fill="currentColor">
                <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 17.93c-3.95-.49-7-3.85-7-7.93 0-.62.08-1.21.21-1.79L9 15v1c0 1.1.9 2 2 2v1.93zm6.9-2.54c-.26-.81-1-1.39-1.9-1.39h-1v-3c0-.55-.45-1-1-1H8v-2h2c.55 0 1-.45 1-1V7h2c1.1 0 2-.9 2-2v-.41c2.93 1.19 5 4.06 5 7.41 0 2.08-.8 3.97-2.1 5.39z"/>
              </svg>
            </div>
            
            <div class="heart-rate-value">
              <span :class="{ 'pulse': isConnected && heartRate > 0 }">{{ heartRate }}</span>
              <span class="unit">BPM</span>
            </div>
            
            <div v-if="isConnected" class="heart-zone" :style="{ color: currentZone.color }">
              {{ currentZone.name }}
            </div>
            <div v-else class="disconnected-text">
              未连接设备
            </div>
          </div>
        </div>
      </div>

      <!-- Stats Grid -->
      <div v-if="isConnected && heartRateHistory.length > 0" class="stats-grid">
        <div class="stat-card">
          <div class="stat-label">平均心率</div>
          <div class="stat-value">{{ averageHeartRate }}</div>
          <div class="stat-unit">BPM</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">最高心率</div>
          <div class="stat-value" style="color: #e74c3c">{{ maxHeartRate }}</div>
          <div class="stat-unit">BPM</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">最低心率</div>
          <div class="stat-value" style="color: #4ecdc4">{{ minHeartRate }}</div>
          <div class="stat-unit">BPM</div>
        </div>
        <div class="stat-card">
          <div class="stat-label">数据点</div>
          <div class="stat-value">{{ heartRateHistory.length }}</div>
          <div class="stat-unit">个</div>
        </div>
      </div>
    </div>

    <!-- Heart Rate Zone Info -->
    <div v-if="isConnected && heartRate > 0" class="zone-info">
      <div class="zone-header">
        <h3>当前区间</h3>
        <span class="zone-name" :style="{ color: currentZone.color }">{{ currentZone.name }}</span>
      </div>
      <p class="zone-description">{{ currentZone.description }}</p>
      
      <div class="zones-bar">
        <div class="zone-segment" style="background: #95a5a6"></div>
        <div class="zone-segment" style="background: #4ecdc4"></div>
        <div class="zone-segment" style="background: #f39c12"></div>
        <div class="zone-segment" style="background: #e74c3c"></div>
        <div class="zone-segment" style="background: #9b59b6"></div>
        <div class="zone-segment" style="background: #c0392b"></div>
        <div 
          class="zone-indicator" 
          :style="{ left: `${Math.min(100, (heartRate / 200) * 100)}%` }"
        ></div>
      </div>
      
      <div class="zones-labels">
        <span>50</span>
        <span>100</span>
        <span>130</span>
        <span>160</span>
        <span>180</span>
        <span>200+</span>
      </div>
    </div>

    <!-- History Panel (Collapsible) -->
    <transition name="slide">
      <div v-if="heartRateHistory.length > 0" class="history-panel">
        <div class="history-header">
          <h3>历史记录</h3>
          <button @click="emit('toggle-history')" class="btn-close">
            <svg viewBox="0 0 24 24" fill="currentColor">
              <path d="M19 6.41L17.59 5 12 10.59 6.41 5 5 6.41 10.59 12 5 17.59 6.41 19 12 13.41 17.59 19 19 17.59 13.41 12z"/>
            </svg>
          </button>
        </div>
        
        <div class="history-list">
          <div 
            v-for="(item, index) in heartRateHistory.slice().reverse()" 
            :key="index"
            class="history-item"
          >
            <span class="history-time">{{ item.time }}</span>
            <div class="history-bar-container">
              <div 
                class="history-bar" 
                :style="{ 
                  width: `${(item.rate / 200) * 100}%`,
                  background: getHeartRateZone(item.rate).color
                }"
              ></div>
            </div>
            <span class="history-value" :style="{ color: getHeartRateZone(item.rate).color }">
              {{ item.rate }}
            </span>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<style scoped>
.heart-rate-monitor {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.monitor-display {
  margin-bottom: 2rem;
}

.display-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.display-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
}

.btn-history {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.5rem 1rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-history:hover {
  background: rgba(255, 255, 255, 0.2);
}

.btn-history svg {
  width: 20px;
  height: 20px;
}

.heart-display {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
}

.heart-wrapper {
  position: relative;
  width: 280px;
  height: 280px;
}

.heart-bg {
  width: 100%;
  height: 100%;
}

.progress-ring {
  transition: stroke-dashoffset 0.5s ease;
}

.heart-content {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.heart-icon {
  width: 64px;
  height: 64px;
  color: #ff6b6b;
  animation: heartbeat 1.5s ease-in-out infinite;
}

@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  10%, 30% { transform: scale(1.1); }
  20%, 40% { transform: scale(1); }
}

.pulse {
  animation: pulse-animation 1s ease-in-out infinite;
}

@keyframes pulse-animation {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.7; }
}

.disconnected-icon {
  width: 64px;
  height: 64px;
  color: rgba(255, 255, 255, 0.5);
}

.disconnected-icon svg {
  width: 100%;
  height: 100%;
}

.heart-rate-value {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.heart-rate-value span:first-child {
  font-size: 4rem;
  font-weight: 700;
  line-height: 1;
}

.unit {
  font-size: 1rem;
  opacity: 0.8;
  margin-top: 0.25rem;
}

.heart-zone {
  font-size: 1.2rem;
  font-weight: 600;
  padding: 0.5rem 1.5rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 20px;
}

.disconnected-text {
  font-size: 1.2rem;
  opacity: 0.7;
}

/* Stats Grid */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 1rem;
  margin-top: 2rem;
}

.stat-card {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1.5rem;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.stat-label {
  font-size: 0.9rem;
  opacity: 0.8;
  margin-bottom: 0.5rem;
}

.stat-value {
  font-size: 2rem;
  font-weight: 700;
  color: #4ecdc4;
}

.stat-unit {
  font-size: 0.85rem;
  opacity: 0.7;
  margin-top: 0.25rem;
}

/* Zone Info */
.zone-info {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1.5rem;
  margin-top: 2rem;
}

.zone-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.5rem;
}

.zone-header h3 {
  font-size: 1.1rem;
  font-weight: 600;
}

.zone-name {
  font-size: 1.2rem;
  font-weight: 700;
}

.zone-description {
  opacity: 0.9;
  margin-bottom: 1.5rem;
}

.zones-bar {
  position: relative;
  height: 12px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 6px;
  overflow: hidden;
  margin-bottom: 0.5rem;
}

.zone-segment {
  float: left;
  height: 100%;
  width: 16.66%;
}

.zone-indicator {
  position: absolute;
  top: -4px;
  width: 4px;
  height: 20px;
  background: white;
  border-radius: 2px;
  transform: translateX(-50%);
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.8);
}

.zones-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.75rem;
  opacity: 0.7;
}

/* History Panel */
.history-panel {
  margin-top: 2rem;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  padding: 1.5rem;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1rem;
}

.history-header h3 {
  font-size: 1.1rem;
  font-weight: 600;
}

.btn-close {
  background: none;
  border: none;
  color: white;
  cursor: pointer;
  padding: 0.5rem;
  border-radius: 50%;
  transition: background 0.3s ease;
}

.btn-close:hover {
  background: rgba(255, 255, 255, 0.1);
}

.btn-close svg {
  width: 20px;
  height: 20px;
}

.history-list {
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
  max-height: 300px;
  overflow-y: auto;
}

.history-item {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 0.75rem;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 8px;
}

.history-time {
  font-size: 0.85rem;
  opacity: 0.8;
  min-width: 60px;
}

.history-bar-container {
  flex: 1;
  height: 8px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  overflow: hidden;
}

.history-bar {
  height: 100%;
  border-radius: 4px;
  transition: width 0.3s ease;
}

.history-value {
  font-weight: 600;
  min-width: 40px;
  text-align: right;
}

/* Slide Transition */
.slide-enter-active,
.slide-leave-active {
  transition: all 0.3s ease;
  max-height: 400px;
  opacity: 1;
}

.slide-enter-from,
.slide-leave-to {
  max-height: 0;
  opacity: 0;
}

/* Responsive Design */
@media (max-width: 768px) {
  .heart-rate-monitor {
    padding: 1.5rem;
  }
  
  .heart-wrapper {
    width: 220px;
    height: 220px;
  }
  
  .heart-rate-value span:first-child {
    font-size: 3rem;
  }
  
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>
