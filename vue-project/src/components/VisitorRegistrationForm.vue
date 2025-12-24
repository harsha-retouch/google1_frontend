<script setup>
import { ref, reactive, computed } from 'vue'

const currentStep = ref(1)
const totalSteps = 4

const formData = reactive({
  // Step 1: Identity
  fullName: '',
  email: '',
  contactNumber: '',
  company: '',
  
  // Step 2: Visit Details
  purpose: 'Meeting',
  hostName: '',
  department: 'Technology',
  expectedDuration: '1-2 Hours',
  
  // Step 3: Security & Assets
  laptopModel: '',
  serialNumber: '',
  needsWifi: false,
  facePhoto: null,
  idType: 'Aadhaar Card',
  idPhoto: null,
  
  // Step 4: Submission
  isAgreed: false,
  isSubmitting: false
})

// UI States
const isVerifyingDoc = ref(false)
const docVerified = ref(false)
const capturedImage = ref(null)
const showCamera = ref(false)
const videoRef = ref(null)
const canvasRef = ref(null)
const stream = ref(null)

// Step Management
const nextStep = () => {
  if (currentStep.value < totalSteps) currentStep.value++
}
const prevStep = () => {
  if (currentStep.value > 1) currentStep.value--
}

// Logic: Camera
const startCamera = async () => {
  try {
    showCamera.value = true
    stream.value = await navigator.mediaDevices.getUserMedia({ 
      video: { facingMode: 'user', width: 640, height: 480 } 
    })
    if (videoRef.value) videoRef.value.srcObject = stream.value
  } catch (err) {
    alert("Camera access denied.")
    showCamera.value = false
  }
}

const stopCamera = () => {
  if (stream.value) {
    stream.value.getTracks().forEach(track => track.stop())
    stream.value = null
  }
  showCamera.value = false
}

const capturePhoto = () => {
  const video = videoRef.value
  const canvas = canvasRef.value
  if (video && canvas) {
    canvas.width = video.videoWidth
    canvas.height = video.videoHeight
    canvas.getContext('2d').drawImage(video, 0, 0)
    const dataUrl = canvas.toDataURL('image/jpeg')
    capturedImage.value = dataUrl
    fetch(dataUrl).then(res => res.blob()).then(blob => {
      formData.facePhoto = new File([blob], "face.jpg", { type: "image/jpeg" })
    })
    stopCamera()
  }
}

// Logic: Doc Verify
const handleDocUpload = async (event) => {
  const file = event.target.files[0]
  if (!file) return
  formData.idPhoto = file
  isVerifyingDoc.value = true
  docVerified.value = false
  
  // Simulated AI Verification
  await new Promise(r => setTimeout(r, 2000))
  docVerified.value = true
  isVerifyingDoc.value = false
}

const handleSubmit = async () => {
  formData.isSubmitting = true
  await new Promise(r => setTimeout(r, 2000))
  alert("RIS International Protocol Successful. Welcome aboard.")
  formData.isSubmitting = false
}

const progressWidth = computed(() => `${(currentStep.value / totalSteps) * 100}%`)
</script>

<template>
  <div class="ris-portal">
    <div class="background-mesh"></div>
    
    <div class="portal-container">
      <header class="portal-header">
        <div class="portal-logo">
          <svg viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 2L2 7L12 12L22 7L12 2Z" fill="#3B82F6"/>
            <path d="M2 17L12 22L22 17" stroke="#8b5cf6" stroke-width="2"/>
            <path d="M2 12L12 17L22 12" stroke="#3B82F6" stroke-width="2"/>
          </svg>
        </div>
        <div class="brand">
          <h1>RIS <span>INTERNATIONAL</span></h1>
          <p>VISITOR PROTOCOL ANALYTICS v.2.4</p>
        </div>
      </header>

      <div class="stepper-bar">
        <div class="progress-track">
          <div class="progress-fill" :style="{ width: progressWidth }"></div>
        </div>
        <div class="step-labels">
          <span :class="{active: currentStep >= 1}">PROFILE</span>
          <span :class="{active: currentStep >= 2}">INTENT</span>
          <span :class="{active: currentStep >= 3}">SECURITY</span>
          <span :class="{active: currentStep >= 4}">VERIFY</span>
        </div>
      </div>

      <main class="portal-content">
        <transition name="slide-fade" mode="out-in">
          <!-- STEP 1: IDENTITY -->
          <div v-if="currentStep === 1" key="step1" class="step-card">
            <h2>Identity Profile</h2>
            <div class="input-grid">
              <div class="neo-field">
                <label>Full Name</label>
                <input type="text" v-model="formData.fullName" placeholder="Enter Legal Name">
              </div>
              <div class="neo-field">
                <label>Corporate Email</label>
                <input type="email" v-model="formData.email" placeholder="name@company.com">
              </div>
              <div class="neo-field">
                <label>Phone Node</label>
                <input type="tel" v-model="formData.contactNumber" placeholder="+91 XXXXX XXXXX">
              </div>
              <div class="neo-field">
                <label>Origin Company</label>
                <input type="text" v-model="formData.company" placeholder="Organization Name">
              </div>
            </div>
          </div>

          <!-- STEP 2: INTENT -->
          <div v-else-if="currentStep === 2" key="step2" class="step-card">
            <h2>Visit Intent</h2>
            <div class="input-grid">
              <div class="neo-field">
                <label>Purpose of Visit</label>
                <select v-model="formData.purpose">
                  <option>Meeting</option>
                  <option>Interview</option>
                  <option>Maintenance</option>
                  <option>Delivery</option>
                  <option>Personal</option>
                </select>
              </div>
              <div class="neo-field">
                <label>Host Name (RIS Personnel)</label>
                <input type="text" v-model="formData.hostName" placeholder="Who are you meeting?">
              </div>
              <div class="neo-field">
                <label>Target Department</label>
                <select v-model="formData.department">
                  <option>Technology</option>
                  <option>Operations</option>
                  <option>Executive</option>
                  <option>Finance</option>
                  <option>HR</option>
                </select>
              </div>
              <div class="neo-field">
                <label>Expected Duration</label>
                <select v-model="formData.expectedDuration">
                  <option>&lt; 1 Hour</option>
                  <option>1-2 Hours</option>
                  <option>Full Day</option>
                </select>
              </div>
            </div>
          </div>

          <!-- STEP 3: SECURITY & ASSETS -->
          <div v-else-if="currentStep === 3" key="step3" class="step-card">
            <h2>Asset & Bio-Identity</h2>
            <div class="security-grid">
              <div class="asset-declaration">
                <h3>Asset Declaration</h3>
                <div class="neo-field">
                  <label>Hardware Model (Laptop/Tablet)</label>
                  <input type="text" v-model="formData.laptopModel" placeholder="e.g. MacBook Pro M2">
                </div>
                <div class="neo-field">
                  <label>Serial Signature</label>
                  <input type="text" v-model="formData.serialNumber" placeholder="Serial Code">
                </div>
                <div class="wifi-toggle">
                  <span>Require WiFi Credentials?</span>
                  <label class="switch">
                    <input type="checkbox" v-model="formData.needsWifi">
                    <span class="slider"></span>
                  </label>
                </div>
              </div>

              <div class="bio-capture">
                <h3>Vibe Capture (Photo)</h3>
                <div class="capture-zone">
                  <div v-if="!showCamera && !capturedImage" class="capture-placeholder" @click="startCamera">
                    <svg viewBox="0 0 24 24" width="40" height="40" stroke="currentColor" stroke-width="2" fill="none"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"></path><circle cx="12" cy="13" r="4"></circle></svg>
                    <p>Start Bio-Scan</p>
                  </div>
                  <div v-if="showCamera" class="live-scan">
                    <video ref="videoRef" autoplay playsinline></video>
                    <button @click="capturePhoto" class="shutter-btn"></button>
                  </div>
                  <div v-if="capturedImage" class="review-scan">
                    <img :src="capturedImage" alt="Bio-Face">
                    <button @click="capturedImage = null; startCamera()" class="mini-btn">Retake</button>
                  </div>
                </div>
              </div>
            </div>
            <canvas ref="canvasRef" style="display:none"></canvas>
          </div>

          <!-- STEP 4: VERIFY -->
          <div v-else-if="currentStep === 4" key="step4" class="step-card">
            <h2>Legal Protocols</h2>
            <div class="doc-verify-box">
              <div class="doc-selector">
                <label>Select Evidence Type</label>
                <select v-model="formData.idType">
                  <option>Aadhaar Card</option>
                  <option>Passport</option>
                  <option>Driving License</option>
                  <option>Company ID Card</option>
                </select>
              </div>
              
              <div class="verify-upload-area" :class="{ 'is-verified': docVerified }">
                <input type="file" id="docFile" @change="handleDocUpload" style="display:none">
                <label for="docFile" class="upload-trigger">
                  <template v-if="!docVerified && !isVerifyingDoc">
                    <svg viewBox="0 0 24 24" width="30" height="30" stroke="#60a5fa" stroke-width="2" fill="none"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>
                    <p>Sync {{ formData.idType }} Image</p>
                  </template>
                  <template v-if="isVerifyingDoc">
                    <div class="scanner-anim"></div>
                    <p>Analyzing Credentials...</p>
                  </template>
                  <template v-if="docVerified">
                    <svg viewBox="0 0 24 24" width="30" height="30" stroke="#4ade80" stroke-width="2" fill="none"><path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"></path><polyline points="22 4 12 14.01 9 11.01"></polyline></svg>
                    <p>Identity Authentication Secure</p>
                  </template>
                </label>
              </div>
            </div>

            <div class="legal-accept">
              <label class="check-container">
                <input type="checkbox" v-model="formData.isAgreed">
                <span class="checkmark"></span>
                I acknowledge the RIS International security protocols, data privacy terms, and non-disclosure requirements.
              </label>
            </div>
          </div>
        </transition>
      </main>

      <footer class="portal-footer">
        <button v-if="currentStep > 1" @click="prevStep" class="nav-btn prev">Back</button>
        <button v-if="currentStep < totalSteps" @click="nextStep" class="nav-btn next" :disabled="currentStep === 1 && !formData.fullName">Next Phase</button>
        <button v-if="currentStep === totalSteps" @click="handleSubmit" class="nav-btn submit" :disabled="!formData.isAgreed || !docVerified || formData.isSubmitting">
          <span v-if="!formData.isSubmitting">Initialize Protocol</span>
          <span v-else class="loader"></span>
        </button>
      </footer>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=Outfit:wght@300;400;600;900&display=swap');

.ris-portal {
  font-family: 'Space Grotesk', sans-serif;
  min-height: 100vh;
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  background: #050505;
  color: #fff;
  overflow-x: hidden;
  padding: 20px;
}

.background-mesh {
  position: fixed;
  top: 0; left: 0; width: 100%; height: 100%;
  background: 
    radial-gradient(circle at 10% 20%, rgba(59, 130, 246, 0.1) 0%, transparent 40%),
    radial-gradient(circle at 90% 80%, rgba(139, 92, 246, 0.1) 0%, transparent 40%),
    radial-gradient(circle at 50% 50%, rgba(0, 0, 0, 1) 0%, #050505 100%);
  z-index: 0;
}

.portal-container {
  position: relative;
  z-index: 1;
  width: 100%;
  max-width: 900px;
  background: rgba(20, 20, 20, 0.6);
  backdrop-filter: blur(40px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 40px;
  padding: 50px;
  box-shadow: 0 40px 100px rgba(0, 0, 0, 0.8);
}

.portal-header {
  display: flex;
  align-items: center;
  gap: 25px;
  margin-bottom: 40px;
}

.portal-logo svg {
  width: 60px;
  height: 60px;
  filter: drop-shadow(0 0 10px rgba(59, 130, 246, 0.5));
}

.brand h1 {
  font-family: 'Outfit', sans-serif;
  font-size: 2.2rem;
  font-weight: 900;
  letter-spacing: -1px;
}

.brand h1 span {
  color: #3b82f6;
  font-weight: 300;
  letter-spacing: 5px;
}

.brand p {
  font-size: 0.75rem;
  letter-spacing: 3px;
  opacity: 0.5;
  color: #60a5fa;
  margin-top: 5px;
}

/* Stepper */
.stepper-bar {
  margin-bottom: 50px;
}

.progress-track {
  height: 4px;
  background: rgba(255, 255, 255, 0.05);
  border-radius: 2px;
  margin-bottom: 15px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(to right, #3b82f6, #8b5cf6);
  transition: width 0.6s cubic-bezier(0.16, 1, 0.3, 1);
  box-shadow: 0 0 20px rgba(59, 130, 246, 0.5);
}

.step-labels {
  display: flex;
  justify-content: space-between;
}

.step-labels span {
  font-size: 0.7rem;
  font-weight: 600;
  letter-spacing: 2px;
  color: rgba(255, 255, 255, 0.3);
  transition: 0.3s;
}

.step-labels span.active {
  color: #3b82f6;
  text-shadow: 0 0 10px rgba(59, 130, 246, 0.3);
}

/* Content Area */
.step-card h2 {
  font-family: 'Outfit', sans-serif;
  font-size: 1.8rem;
  margin-bottom: 30px;
  font-weight: 600;
}

.input-grid {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 25px;
}

.neo-field {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.neo-field label {
  font-size: 0.85rem;
  font-weight: 500;
  color: #60a5fa;
  opacity: 0.8;
}

.neo-field input, 
.neo-field select {
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 16px;
  padding: 16px 20px;
  color: #fff;
  font-size: 1rem;
  font-family: inherit;
  transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
}

.neo-field input:focus, 
.neo-field select:focus {
  outline: none;
  background: rgba(59, 130, 246, 0.05);
  border-color: #3b82f6;
  box-shadow: 0 0 30px rgba(59, 130, 246, 0.1);
}

/* Security Grid */
.security-grid {
  display: grid;
  grid-template-columns: 1.5fr 1fr;
  gap: 40px;
}

.asset-declaration h3, 
.bio-capture h3 {
  font-size: 1rem;
  color: #60a5fa;
  margin-bottom: 20px;
  font-weight: 600;
}

.wifi-toggle {
  margin-top: 25px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: rgba(255, 255, 255, 0.03);
  padding: 15px 20px;
  border-radius: 16px;
}

.wifi-toggle span {
  font-size: 0.9rem;
}

.capture-zone {
  aspect-ratio: 1;
  background: rgba(0, 0, 0, 0.3);
  border: 2px dashed rgba(255, 255, 255, 0.1);
  border-radius: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  position: relative;
  cursor: pointer;
  transition: 0.3s;
}

.capture-zone:hover {
  border-color: #3b82f6;
  background: rgba(59, 130, 246, 0.05);
}

.capture-placeholder {
  text-align: center;
  color: #60a5fa;
}

.live-scan video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.shutter-btn {
  position: absolute;
  bottom: 20px;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: #fff;
  border: 5px solid rgba(59, 130, 246, 0.3);
  cursor: pointer;
}

.review-scan img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.mini-btn {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background: rgba(0, 0, 0, 0.6);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 5px 12px;
  border-radius: 8px;
  font-size: 0.75rem;
}

/* Verification */
.doc-verify-box {
  background: rgba(255, 255, 255, 0.02);
  padding: 30px;
  border-radius: 24px;
  border: 1px solid rgba(255, 255, 255, 0.05);
}

.doc-selector {
  margin-bottom: 25px;
}

.doc-selector select {
  width: 100%;
  background: #111;
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 12px;
  border-radius: 12px;
  color: white;
}

.verify-upload-area {
  height: 120px;
  border: 2px dashed rgba(59, 130, 246, 0.3);
  border-radius: 16px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: 0.3s;
}

.verify-upload-area.is-verified {
  border-color: #4ade80;
  background: rgba(74, 222, 128, 0.05);
}

.upload-trigger {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  gap: 10px;
}

.scanner-anim {
  width: 40px;
  height: 2px;
  background: #3b82f6;
  box-shadow: 0 0 15px #3b82f6;
  animation: scanH 2s infinite ease-in-out;
}

@keyframes scanH {
  0%, 100% { transform: translateY(-20px); }
  50% { transform: translateY(20px); }
}

.legal-accept {
  margin-top: 30px;
}

/* Footers Buttons */
.portal-footer {
  margin-top: 50px;
  display: flex;
  justify-content: space-between;
  gap: 20px;
}

.nav-btn {
  padding: 16px 35px;
  border-radius: 20px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.4s cubic-bezier(0.16, 1, 0.3, 1);
  font-family: inherit;
}

.prev {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.1);
  color: #fff;
}

.next {
  background: #fff;
  border: none;
  color: #000;
  flex-grow: 1;
}

.submit {
  background: linear-gradient(135deg, #3b82f6, #8b5cf6);
  border: none;
  color: #fff;
  flex-grow: 1;
  box-shadow: 0 10px 30px rgba(59,130,246,0.3);
}

.nav-btn:hover:not(:disabled) {
  transform: translateY(-4px);
  filter: brightness(1.1);
}

.nav-btn:disabled {
  opacity: 0.3;
  cursor: not-allowed;
  transform: none;
}

/* Transitions */
.slide-fade-enter-active { transition: all 0.5s ease; }
.slide-fade-leave-active { transition: all 0.4s ease; }
.slide-fade-enter-from { opacity: 0; transform: translateX(30px); }
.slide-fade-leave-to { opacity: 0; transform: translateX(-30px); }

/* Switch Toggle */
.switch { position: relative; display: inline-block; width: 50px; height: 26px; }
.switch input { opacity: 0; width: 0; height: 0; }
.slider {
  position: absolute; cursor: pointer; top: 0; left: 0; right: 0; bottom: 0;
  background-color: #333; transition: .4s; border-radius: 34px;
}
.slider:before {
  position: absolute; content: ""; height: 18px; width: 18px; left: 4px; bottom: 4px;
  background-color: white; transition: .4s; border-radius: 50%;
}
input:checked + .slider { background-color: #3b82f6; }
input:checked + .slider:before { transform: translateX(24px); }

/* Responsive */
@media (max-width: 768px) {
  .portal-container { padding: 30px; border-radius: 0; min-height: 100vh; }
  .input-grid, .security-grid { grid-template-columns: 1fr; }
  .portal-header { flex-direction: column; text-align: center; }
  .nav-btn { padding: 16px 20px; }
}
</style>
