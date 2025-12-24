<script setup>
import { ref } from 'vue'
import VisitorRegistrationForm from './components/VisitorRegistrationForm.vue'

const showForm = ref(false)
const showLinkPopup = ref(false)

// CHANGE THIS: Replace with your computer's local IP address (e.g., '192.168.1.10')
// You can find your IP by running 'ipconfig' in your terminal
const LOCAL_IP = '127.0.0.1' 
const APP_PORT = '5173'
const registrationUrl = `http://${LOCAL_IP}:${APP_PORT}`

const handleScan = () => {
  showLinkPopup.value = true
}

const goToForm = () => {
  showForm.value = true
  showLinkPopup.value = false
}
</script>

<template>
  <main>
    <div v-if="!showForm" class="scanner-view">
      <div class="sky-background"></div>
      
      <div class="scanner-content">
        <div class="brand-logo">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 2L2 7L12 12L22 7L12 2Z" fill="url(#grad1)"/>
            <path d="M2 17L12 22L22 17" stroke="url(#grad1)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <path d="M2 12L12 17L22 12" stroke="url(#grad1)" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            <defs>
              <linearGradient id="grad1" x1="2" y1="2" x2="22" y2="22" gradientUnits="userSpaceOnUse">
                <stop stop-color="#3b82f6" />
                <stop offset="1" stop-color="#8b5cf6" />
              </linearGradient>
            </defs>
          </svg>
        </div>
        <h1>RIS INTERNATIONAL</h1>
        <p class="subtitle">Next-Gen Corporate Visitor Protocol</p>
        
        <div class="qr-container" @click="handleScan">
          <img :src="`https://api.qrserver.com/v1/create-qr-code/?size=300x300&data=${registrationUrl}&color=3b82f6`" alt="QR Code" class="qr-image">
          <div class="scan-line"></div>
          <p class="tap-hint">VIBE CHECK TO START</p>
        </div>
      </div>

      <Transition name="fade-up">
        <div v-if="showLinkPopup" class="link-popup">
          <div class="link-card">
            <div class="icon">
              <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"><path d="M10 13a5 5 0 0 0 7.54.54l3-3a5 5 0 0 0-7.07-7.07l-1.72 1.71"></path><path d="M14 11a5 5 0 0 0-7.54-.54l-3 3a5 5 0 0 0 7.07 7.07l1.71-1.71"></path></svg>
            </div>
            <div class="text">
              <h4>QR Code Detected</h4>
              <p>{{ registrationUrl }}</p>
            </div>
            <button @click="goToForm" class="open-btn">Open Form</button>
          </div>
        </div>
      </Transition>
    </div>

    <Transition name="page-transition">
      <VisitorRegistrationForm v-if="showForm" />
    </Transition>
  </main>
</template>

<style>
/* Global resets */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: #0f172a;
  overflow-x: hidden;
}

.sky-background {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: #000;
  background-image: 
    radial-gradient(at 0% 0%, hsla(222,47%,11%,1) 0, transparent 50%), 
    radial-gradient(at 50% 0%, hsla(220,100%,15%,1) 0, transparent 50%), 
    radial-gradient(at 100% 0%, hsla(240,65%,15%,1) 0, transparent 50%);
  z-index: -1;
}

.scanner-view {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  color: white;
  font-family: 'Outfit', sans-serif;
  text-align: center;
}

.brand-logo {
  margin-bottom: 20px;
  animation: pulse 2s infinite ease-in-out;
}

.brand-logo svg {
  width: 80px;
  height: 80px;
}

@keyframes pulse {
  0% { transform: scale(1); filter: drop-shadow(0 0 0px #3b82f6); }
  50% { transform: scale(1.1); filter: drop-shadow(0 0 20px #3b82f6); }
  100% { transform: scale(1); filter: drop-shadow(0 0 0px #3b82f6); }
}

.scanner-content h1 {
  font-size: 3rem;
  letter-spacing: 4px;
  font-weight: 900;
  margin-bottom: 5px;
  background: linear-gradient(135deg, #fff 0%, #3b82f6 50%, #8b5cf6 100%);
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 0 10px 30px rgba(59, 130, 246, 0.3);
}

.subtitle {
  font-size: 1rem;
  font-weight: 300;
  letter-spacing: 2px;
  text-transform: uppercase;
  color: #60a5fa;
  margin-bottom: 40px;
  opacity: 0.8;
}

.qr-container {
  position: relative;
  width: 280px;
  height: 280px;
  background: rgba(255, 255, 255, 0.05);
  border: 4px solid rgba(255, 255, 255, 0.1);
  border-radius: 40px;
  padding: 30px;
  cursor: pointer;
  transition: transform 0.3s ease;
  overflow: hidden;
}

.qr-container:hover {
  transform: scale(1.02);
  border-color: rgba(96, 165, 250, 0.3);
}

.qr-image {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 20px;
}

.scan-line {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 2px;
  background: #3b82f6;
  box-shadow: 0 0 15px #3b82f6, 0 0 30px #3b82f6;
  animation: scan 3s infinite linear;
}

@keyframes scan {
  0% { top: 10%; }
  50% { top: 90%; }
  100% { top: 10%; }
}

.tap-hint {
  margin-top: 20px;
  font-size: 0.8rem;
  opacity: 0.5;
  text-transform: uppercase;
  letter-spacing: 1px;
}

/* Link Popup */
.link-popup {
  position: fixed;
  bottom: 40px;
  left: 50%;
  transform: translateX(-50%);
  width: calc(100% - 40px);
  max-width: 400px;
  z-index: 100;
}

.link-card {
  background: rgba(255, 255, 255, 0.1);
  backdrop-filter: blur(20px);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 24px;
  padding: 16px 20px;
  display: flex;
  align-items: center;
  gap: 15px;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}

.link-card .icon {
  background: #3b82f6;
  width: 40px;
  height: 40px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.link-card .text {
  text-align: left;
  flex: 1;
}

.link-card h4 {
  font-size: 0.9rem;
  margin: 0;
}

.link-card p {
  font-size: 0.75rem;
  opacity: 0.6;
  margin: 0;
}

.open-btn {
  background: white;
  color: black;
  border: none;
  border-radius: 12px;
  padding: 10px 16px;
  font-weight: 600;
  font-size: 0.85rem;
  cursor: pointer;
}

/* Transitions */
.fade-up-enter-active,
.fade-up-leave-active {
  transition: all 0.5s cubic-bezier(0.16, 1, 0.3, 1);
}

.fade-up-enter-from {
  opacity: 0;
  transform: translate(-50%, 20px);
}

.fade-up-leave-to {
  opacity: 0;
  transform: translate(-50%, 20px);
}

.page-transition-enter-active {
  transition: all 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

.page-transition-enter-from {
  opacity: 0;
  transform: scale(1.1);
  filter: blur(10px);
}

@media (max-width: 480px) {
  .scanner-content h1 {
    font-size: 1.6rem;
  }
}
</style>
