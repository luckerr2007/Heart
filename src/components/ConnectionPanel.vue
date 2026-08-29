<script setup>
import { ref, onMounted, onUnmounted } from 'vue'

const props = defineProps({
  isConnected: Boolean,
  deviceName: String
})

const emit = defineEmits(['connect'])

const isConnecting = ref(false)
const availableDevices = ref([])
const error = ref('')

// Simulated devices for demo
const simulatedDevices = [
  { id: 1, name: 'Polar H10', type: '蓝牙心率带' },
  { id: 2, name: 'Garmin HRM-Pro', type: '蓝牙心率带' },
  { id: 3, name: 'Wahoo TICKR', type: '蓝牙心率带' },
  { id: 4, name: 'Suunto Smart Sensor', type: '蓝牙心率带' }
]

let heartRateSimulation = null
let currentHeartRate = 70

const scanDevices = async () => {
  isConnecting.value = true
  error.value = ''
  
  try {
    // Check if Web Bluetooth API is available
    if (navigator.bluetooth) {
      // Request Bluetooth device with Heart Rate Service
      const device = await navigator.bluetooth.requestDevice({
        filters: [{ services: ['heart_rate'] }]
      })
      
      connectToDevice(device)
    } else {
      // Fallback to simulated devices
      availableDevices.value = simulatedDevices
      isConnecting.value = false
    }
  } catch (err) {
    if (err.name === 'NotFoundError') {
      error.value = '未选择设备'
    } else if (err.name === 'SecurityError') {
      error.value = '蓝牙权限被拒绝'
    } else {
      // Use simulated mode for demo
      availableDevices.value = simulatedDevices
      error.value = 'Web Bluetooth API 不可用，使用模拟模式'
    }
    isConnecting.value = false
  }
}

const connectToDevice = async (device) => {
  try {
    const server = await device.gatt.connect()
    const service = await server.getPrimaryService('heart_rate')
    const characteristic = await service.getCharacteristic('heart_rate_measurement')
    
    characteristic.addEventListener('characteristicvaluechanged', handleHeartRateChange)
    await characteristic.startNotifications()
    
    emit('connect', true, device.name)
    
    device.addEventListener('gattserverdisconnected', handleDisconnect)
  } catch (err) {
    error.value = '连接失败：' + err.message
  } finally {
    isConnecting.value = false
  }
}

const handleHeartRateChange = (event) => {
  const value = event.target.value
  let heartRate = 0
  
  // Parse heart rate measurement
  const flags = value.getUint8(0)
  const format = flags & 0x01 ? 16 : 8
  
  if (format === 8) {
    heartRate = value.getUint8(1)
  } else {
    heartRate = value.getUint16(1, true)
  }
  
  // Emit heart rate to parent
  // This would be handled by the parent component
}

const handleDisconnect = () => {
  emit('connect', false, '')
  if (heartRateSimulation) {
    clearInterval(heartRateSimulation)
  }
}

const connectSimulatedDevice = (device) => {
  isConnecting.value = true
  
  setTimeout(() => {
    emit('connect', true, device.name)
    isConnecting.value = false
    
    // Simulate heart rate changes
    currentHeartRate = 70
    heartRateSimulation = setInterval(() => {
      // Simulate realistic heart rate variation
      const change = Math.floor(Math.random() * 5) - 2
      currentHeartRate = Math.max(60, Math.min(180, currentHeartRate + change))
      
      // Dispatch custom event with heart rate
      window.dispatchEvent(new CustomEvent('heart-rate-update', { 
        detail: { rate: currentHeartRate } 
      }))
    }, 1000)
  }, 1000)
}

const disconnect = () => {
  emit('connect', false, '')
  if (heartRateSimulation) {
    clearInterval(heartRateSimulation)
    heartRateSimulation = null
  }
}

onUnmounted(() => {
  if (heartRateSimulation) {
    clearInterval(heartRateSimulation)
  }
})
</script>

<template>
  <div class="connection-panel">
    <div class="panel-header">
      <h2>设备连接</h2>
      <p>连接蓝牙心率带或选择模拟设备</p>
    </div>

    <div class="panel-content">
      <!-- Connection Status -->
      <div v-if="isConnected" class="status-connected">
        <div class="status-indicator connected"></div>
        <div class="status-info">
          <span class="status-text">已连接</span>
          <span class="device-name">{{ deviceName }}</span>
        </div>
        <button @click="disconnect" class="btn-disconnect">断开</button>
      </div>

      <!-- Scan Button -->
      <div v-else class="scan-section">
        <button 
          @click="scanDevices" 
          class="btn-scan"
          :disabled="isConnecting"
        >
          <svg v-if="!isConnecting" viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm-1 17.93c-3.95-.49-7-3.85-7-7.93 0-.62.08-1.21.21-1.79L9 15v1c0 1.1.9 2 2 2v1.93zm6.9-2.54c-.26-.81-1-1.39-1.9-1.39h-1v-3c0-.55-.45-1-1-1H8v-2h2c.55 0 1-.45 1-1V7h2c1.1 0 2-.9 2-2v-.41c2.93 1.19 5 4.06 5 7.41 0 2.08-.8 3.97-2.1 5.39z"/>
          </svg>
          <svg v-else class="spinner" viewBox="0 0 24 24" fill="none">
            <circle cx="12" cy="12" r="10" stroke="currentColor" stroke-width="3" stroke-dasharray="31.4 31.4" stroke-linecap="round"/>
          </svg>
          {{ isConnecting ? '扫描中...' : '扫描设备' }}
        </button>

        <!-- Available Devices -->
        <div v-if="availableDevices.length > 0" class="devices-list">
          <h3>可用设备</h3>
          <div class="device-grid">
            <button
              v-for="device in availableDevices"
              :key="device.id"
              @click="connectSimulatedDevice(device)"
              class="device-card"
            >
              <div class="device-icon">
                <svg viewBox="0 0 24 24" fill="currentColor">
                  <path d="M18.75 12.75h1.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75h-1.5c-.41 0-.75.34-.75.75s.34.75.75.75zm0 4h1.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75h-1.5c-.41 0-.75.34-.75.75s.34.75.75.75zM6.75 12h1.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75h-1.5c-.41 0-.75.34-.75.75s.34.75.75.75zm0-4h1.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75h-1.5c-.41 0-.75.34-.75.75s.34.75.75.75zm0 8h1.5c.41 0 .75-.34.75-.75s-.34-.75-.75-.75h-1.5c-.41 0-.75.34-.75.75s.34.75.75.75zM12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.42 0-8-3.58-8-8s3.58-8 8-8 8 3.58 8 8-3.58 8-8 8z"/>
                </svg>
              </div>
              <div class="device-info">
                <span class="device-name">{{ device.name }}</span>
                <span class="device-type">{{ device.type }}</span>
              </div>
            </button>
          </div>
        </div>

        <!-- Error Message -->
        <div v-if="error" class="error-message">
          <svg viewBox="0 0 24 24" fill="currentColor">
            <path d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm1 15h-2v-2h2v2zm0-4h-2V7h2v6z"/>
          </svg>
          {{ error }}
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.connection-panel {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(10px);
  border-radius: 16px;
  padding: 2rem;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.panel-header {
  margin-bottom: 1.5rem;
}

.panel-header h2 {
  font-size: 1.5rem;
  font-weight: 600;
  margin-bottom: 0.5rem;
}

.panel-header p {
  opacity: 0.9;
  font-size: 0.95rem;
}

.status-connected {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1.5rem;
  background: rgba(78, 205, 196, 0.2);
  border-radius: 12px;
  border: 1px solid rgba(78, 205, 196, 0.3);
}

.status-indicator {
  width: 12px;
  height: 12px;
  border-radius: 50%;
  animation: pulse 2s ease-in-out infinite;
}

.status-indicator.connected {
  background: #4ecdc4;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.status-info {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.status-text {
  font-weight: 600;
  color: #4ecdc4;
}

.device-name {
  font-size: 0.9rem;
  opacity: 0.9;
}

.btn-disconnect {
  padding: 0.5rem 1.5rem;
  background: rgba(255, 107, 107, 0.8);
  border: none;
  border-radius: 8px;
  color: white;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-disconnect:hover {
  background: rgba(255, 107, 107, 1);
  transform: translateY(-2px);
}

.scan-section {
  display: flex;
  flex-direction: column;
  gap: 1.5rem;
}

.btn-scan {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  width: 100%;
  padding: 1rem 2rem;
  background: linear-gradient(135deg, #4ecdc4 0%, #44a08d 100%);
  border: none;
  border-radius: 12px;
  color: white;
  font-size: 1.1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn-scan:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 10px 30px rgba(78, 205, 196, 0.4);
}

.btn-scan:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

.btn-scan svg {
  width: 24px;
  height: 24px;
}

.spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.devices-list h3 {
  font-size: 1.1rem;
  margin-bottom: 1rem;
  font-weight: 600;
}

.device-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1rem;
}

.device-card {
  display: flex;
  align-items: center;
  gap: 1rem;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
}

.device-card:hover {
  background: rgba(255, 255, 255, 0.2);
  transform: translateY(-2px);
}

.device-icon {
  width: 48px;
  height: 48px;
  color: #4ecdc4;
}

.device-icon svg {
  width: 100%;
  height: 100%;
}

.device-info {
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.device-info .device-name {
  font-weight: 600;
  font-size: 1rem;
}

.device-type {
  font-size: 0.85rem;
  opacity: 0.8;
}

.error-message {
  display: flex;
  align-items: center;
  gap: 0.75rem;
  padding: 1rem;
  background: rgba(255, 107, 107, 0.2);
  border: 1px solid rgba(255, 107, 107, 0.3);
  border-radius: 12px;
  color: #ff6b6b;
}

.error-message svg {
  width: 24px;
  height: 24px;
  flex-shrink: 0;
}

@media (max-width: 768px) {
  .connection-panel {
    padding: 1.5rem;
  }
  
  .device-grid {
    grid-template-columns: 1fr;
  }
}
</style>
