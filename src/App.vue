<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import HeartRateMonitor from './components/HeartRateMonitor.vue'
import ConnectionPanel from './components/ConnectionPanel.vue'

const isConnected = ref(false)
const heartRate = ref(0)
const deviceName = ref('')
const showHistory = ref(false)

const handleConnect = (connected, name) => {
  isConnected.value = connected
  deviceName.value = name
}

const handleHeartRate = (rate) => {
  heartRate.value = rate
}
</script>

<template>
  <div class="app-container">
    <!-- Header -->
    <header class="header">
      <div class="header-content">
        <div class="logo">
          <svg class="heart-icon" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"/>
          </svg>
          <h1>心率广播</h1>
        </div>
        <p class="subtitle">实时心率监测与广播</p>
      </div>
    </header>

    <!-- Main Content -->
    <main class="main-content">
      <div class="content-wrapper">
        <!-- Connection Panel -->
        <ConnectionPanel 
          @connect="handleConnect"
          :isConnected="isConnected"
          :deviceName="deviceName"
        />

        <!-- Heart Rate Monitor -->
        <HeartRateMonitor 
          :heartRate="heartRate"
          :isConnected="isConnected"
          @toggle-history="showHistory = !showHistory"
        />

        <!-- Features Section -->
        <section class="features">
          <div class="feature-card">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="currentColor">
                <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-2 15l-5-5 1.41-1.41L10 14.17l7.59-7.59L19 8l-9 9z"/>
              </svg>
            </div>
            <h3>蓝牙连接</h3>
            <p>支持蓝牙心率带连接，实时获取心率数据</p>
          </div>
          <div class="feature-card">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="currentColor">
                <path d="M12 3v9.28c-.47-.17-.97-.28-1.5-.28C8.01 12 6 14.01 6 16.5S8.01 21 10.5 21c2.31 0 4.2-1.75 4.45-4H15V6h4V3h-7z"/>
              </svg>
            </div>
            <h3>实时广播</h3>
            <p>将心率数据广播到其他设备，支持多平台</p>
          </div>
          <div class="feature-card">
            <div class="feature-icon">
              <svg viewBox="0 0 24 24" fill="currentColor">
                <path d="M3.5 18.49l6-6.01 4 4L22 6.92l-1.41-1.41-7.09 7.97-4-4L2 16.99z"/>
              </svg>
            </div>
            <h3>数据分析</h3>
            <p>记录和分析心率变化趋势，提供健康建议</p>
          </div>
        </section>
      </div>
    </main>

    <!-- Footer -->
    <footer class="footer">
      <p>© 2024 心率广播 | 类似项目：心宿心率广播</p>
    </footer>
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  min-height: 100vh;
  color: #ffffff;
}

.app-container {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

/* Header Styles */
.header {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding: 2rem 0;
}

.header-content {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 2rem;
  text-align: center;
}

.logo {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.heart-icon {
  width: 48px;
  height: 48px;
  color: #ff6b6b;
  animation: heartbeat 1.5s ease-in-out infinite;
}

@keyframes heartbeat {
  0%, 100% { transform: scale(1); }
  10%, 30% { transform: scale(1.1); }
  20%, 40% { transform: scale(1); }
}

.logo h1 {
  font-size: 2.5rem;
  font-weight: 700;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.subtitle {
  font-size: 1.1rem;
  opacity: 0.9;
  font-weight: 300;
}

/* Main Content */
.main-content {
  flex: 1;
  padding: 3rem 2rem;
}

.content-wrapper {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 2rem;
}

/* Features Section */
.features {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
  margin-top: 2rem;
}

.feature-card {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 2rem;
  text-align: center;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.feature-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.3);
}

.feature-icon {
  width: 64px;
  height: 64px;
  margin: 0 auto 1rem;
  color: #4ecdc4;
}

.feature-icon svg {
  width: 100%;
  height: 100%;
}

.feature-card h3 {
  font-size: 1.5rem;
  margin-bottom: 0.5rem;
  font-weight: 600;
}

.feature-card p {
  opacity: 0.9;
  line-height: 1.6;
}

/* Footer */
.footer {
  background: rgba(0, 0, 0, 0.2);
  padding: 2rem;
  text-align: center;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
}

.footer p {
  opacity: 0.8;
  font-size: 0.9rem;
}

/* Responsive Design */
@media (max-width: 768px) {
  .logo h1 {
    font-size: 2rem;
  }
  
  .heart-icon {
    width: 36px;
    height: 36px;
  }
  
  .main-content {
    padding: 2rem 1rem;
  }
  
  .features {
    grid-template-columns: 1fr;
  }
}
</style>
