<template>
  <section class="form-page">
    <div class="card">
      <div class="card-top">
        <img src="./assets/logo.jpg" alt="Logo" class="brand" />
        <h4 class="subtle">Visitor Registration Form</h4>
      </div>

      <h2 class="title">Visitors Details</h2>

      <form @submit.prevent="onSubmit" novalidate>
        <div class="field">
          <label>Full Name <span class="req">*</span></label>
          <input v-model="form.fullName" type="text" placeholder="Full Name" aria-required="true" />
          <small v-if="errors.fullName" class="error">{{ errors.fullName }}</small>
        </div>

        <div class="field">
          <label>Contact Number <span class="req">*</span></label>
          <input v-model="form.contactNumber" type="tel" placeholder="Contact Number" aria-required="true" />
          <small v-if="errors.contactNumber" class="error">{{ errors.contactNumber }}</small>
        </div>

        <div class="two-col">
          <div class="field">
            <label>Gender <span class="req">*</span></label>
            <select v-model="form.gender" aria-required="true">
              <option value="">Select</option>
              <option>Male</option>
              <option>Female</option>
              <option>Other</option>
            </select>
            <small v-if="errors.gender" class="error">{{ errors.gender }}</small>
          </div>

          <div class="field">
            <label>Emergency Contact Number</label>
            <input v-model="form.emergencyNumber" type="tel" placeholder="Emergency Contact Number" />
          </div>
        </div>

        <div class="field">
          <label>Home Town Address <span class="req">*</span></label>
          <textarea v-model="form.address" placeholder="Home Town Address" rows="3" aria-required="true"></textarea>
          <small v-if="errors.address" class="error">{{ errors.address }}</small>
        </div>

        <div class="two-col">
          <div class="field">
            <label>Pin Code <span class="req">*</span></label>
            <input v-model="form.pin" type="text" placeholder="Pin Code" aria-required="true" />
            <small v-if="errors.pin" class="error">{{ errors.pin }}</small>
          </div>

          <div class="field">
            <label>Vehicle Number</label>
            <input v-model="form.vehicle" type="text" placeholder="Vehicle Number" />
          </div>
        </div>

        <div class="field">
          <label>Face Photo <span class="req">*</span></label>
          <div class="file-input-group">
            <input ref="facePhotoInput" @change="onFileChange($event, 'face')" type="file" accept="image/*" class="hidden-input" />
            <button type="button" class="file-btn" @click="triggerFileInput('face')">Choose File</button>
            <button type="button" class="camera-btn" @click="triggerCamera('face')">📷 Camera</button>
            <span v-if="form.faceFile" class="file-name">{{ form.faceFile.name }}</span>
          </div>
          <small class="hint">*(accepted formats - jpg | jpeg | png | gif)</small>
          <small v-if="errors.face" class="error">{{ errors.face }}</small>
        </div>

        <div class="field">
          <label>Aadhaar Document Photo <span class="req">*</span></label>
          <input @change="onFileChange($event, 'aadhar')" type="file" accept="image/*" />
          <small class="hint">*(accepted formats - jpg | jpeg | png | gif)</small>
          <small v-if="errors.aadhar" class="error">{{ errors.aadhar }}</small>
        </div>

        <div class="field">
          <label>Document Type <span class="req">*</span></label>
          <div class="select-row">
            <select v-model="form.docType" aria-required="true" :disabled="loadingFormConfig || docOptions.length === 0">
              <option value="">{{ loadingFormConfig ? 'Loading...' : (docOptions.length ? 'Select' : 'No options available') }}</option>
              <option v-for="opt in docOptions" :key="opt">{{ opt }}</option>
            </select>
            <button type="button" class="btn-secondary btn-small" @click="loadFormConfig" :disabled="loadingFormConfig">Retry</button>
          </div>
          <small v-if="errors.docType" class="error">{{ errors.docType }}</small>
          <small v-if="formConfigError && formConfigError.includes('Using default')" class="hint">{{ formConfigError }}</small>
          <small v-else-if="formConfigError" class="error">{{ formConfigError }}</small>
        </div>  

        <div class="field">
          <label>Aadhaar Number <span class="req">*</span></label>
          <input v-model="form.aadhaarNumber" type="text" placeholder="Aadhaar Number" aria-required="true" />
          <small v-if="errors.aadhaarNumber" class="error">{{ errors.aadhaarNumber }}</small>
        </div>

        <div class="field checkbox-row">
          <label><input type="checkbox" v-model="form.confirm" /> Are you submitting?</label>
        </div>

        <div class="actions">
          <button class="btn" type="submit">Submit</button>
        </div>
      </form>

      <div v-if="lastSavedVisitor" class="saved-section">
        <h3 class="subtle">Saved Visitor (latest)</h3>
        <div class="saved-grid">
          <div><strong>Name:</strong> {{ lastSavedVisitor.full_name || lastSavedVisitor.fullName }}</div>
          <div><strong>Contact:</strong> {{ lastSavedVisitor.contact_number || lastSavedVisitor.contactNumber }}</div>
          <div><strong>Document:</strong> {{ lastSavedVisitor.document_type }}</div>
          <div><strong>Aadhaar:</strong> {{ lastSavedVisitor.aadhaar_number || lastSavedVisitor.aadhaarNumber }}</div>
          <div v-if="lastSavedVisitor.face_photo_url || lastSavedVisitor.facePhotoUrl" class="img-col">
            <strong>Face:</strong>
            <img :src="lastSavedVisitor.face_photo_url || lastSavedVisitor.facePhotoUrl" alt="Face photo" class="thumb" />
          </div>
          <div v-if="lastSavedVisitor.aadhaar_photo_url || lastSavedVisitor.aadhaarPhotoUrl" class="img-col">
            <strong>Aadhaar:</strong>
            <img :src="lastSavedVisitor.aadhaar_photo_url || lastSavedVisitor.aadhaarPhotoUrl" alt="Aadhaar photo" class="thumb" />
          </div>
        </div>
      </div>

    </div>
  </section>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'

const facePhotoInput = ref(null)
const cameraStream = ref(null)

const form = reactive({
  fullName: '',
  contactNumber: '',
  gender: '',
  emergencyNumber: '',
  address: '',
  pin: '',
  vehicle: '',
  faceFile: null,
  aadharFile: null,
  docType: '',
  aadhaarNumber: '',
  confirm: false,
})

const errors = reactive({})
const formConfig = ref(null)
const docOptions = ref([])
const loadingFormConfig = ref(false)
const formConfigError = ref(null)
const lastSavedVisitor = ref(null)

const apiBases = ['', 'http://127.0.0.1:8000', 'http://localhost:8000']

onMounted(() => {
  loadFormConfig()
})

async function loadFormConfig() {
  loadingFormConfig.value = true
  formConfigError.value = null
  docOptions.value = []

  for (const base of apiBases) {
    try {
      const url = `${base}/form-config`
      const res = await fetch(url)
      if (!res.ok) throw new Error(`${res.status} ${res.statusText}`)
      const data = await res.json()
      formConfig.value = data
      const docField = data.fields?.find(f => f.name === 'document_type')
      if (docField && Array.isArray(docField.options) && docField.options.length) {
        docOptions.value = docField.options
      } else {
        // Ensure we have at least a sensible default
        console.warn('document_type field missing or has no options in form-config; using default')
        docOptions.value = ['Aadhaar Card']
      }
      console.log('Loaded form config from', url, data)
      loadingFormConfig.value = false
      return
    } catch (err) {
      console.warn('Could not load form config from', base, err)
      // try next base
    }
  }

  // If we reached here, none of the bases worked
  formConfigError.value = "Could not load form configuration. Using default Document Type 'Aadhaar Card'. Please ensure the backend is running or configure proxy."
  docOptions.value = ['Aadhaar Card']
  loadingFormConfig.value = false
} 

function triggerFileInput(field) {
  if (field === 'face') {
    facePhotoInput.value?.click()
  }
}

function onFileChange(e, field) {
  const file = e.target.files && e.target.files[0]
  if (field === 'face') {
    form.faceFile = file
    console.log('Face photo selected:', file)
  }
  if (field === 'aadhar') form.aadharFile = file
}

function triggerCamera(field) {
  if (field === 'face') {
    // Request camera access
    navigator.mediaDevices.getUserMedia({ video: { facingMode: 'user' } })
      .then(stream => {
        cameraStream.value = stream
        openCameraModal(field)
      })
      .catch(err => {
        console.error('Camera access denied:', err)
        alert('Camera access denied. Please use "Choose File" to upload an image.')
      })
  }
}

function openCameraModal(field) {
  const modal = document.createElement('div')
  modal.className = 'camera-modal'
  modal.innerHTML = `
    <div class="camera-modal-content">
      <h3>Take Photo</h3>
      <video id="camera-video" autoplay playsinline></video>
      <canvas id="camera-canvas" style="display:none;"></canvas>
      <div class="camera-modal-buttons">
        <button type="button" class="btn-secondary" id="cancel-btn">Cancel</button>
        <button type="button" class="btn-primary" id="capture-btn">📸 Capture</button>
      </div>
    </div>
  `
  document.body.appendChild(modal)
  
  const video = modal.querySelector('#camera-video')
  const canvas = modal.querySelector('#camera-canvas')
  const captureBtn = modal.querySelector('#capture-btn')
  const cancelBtn = modal.querySelector('#cancel-btn')
  
  if (cameraStream.value) {
    video.srcObject = cameraStream.value
  }
  
  captureBtn.addEventListener('click', () => {
    canvas.width = video.videoWidth
    canvas.height = video.videoHeight
    const ctx = canvas.getContext('2d')
    ctx.drawImage(video, 0, 0)
    canvas.toBlob(blob => {
      const file = new File([blob], `face_photo_${Date.now()}.jpg`, { type: 'image/jpeg' })
      form.faceFile = file
      stopCamera()
      modal.remove()
      console.log('Photo captured:', file)
    }, 'image/jpeg')
  })
  
  cancelBtn.addEventListener('click', () => {
    stopCamera()
    modal.remove()
  })
}

function stopCamera() {
  if (cameraStream.value) {
    cameraStream.value.getTracks().forEach(track => track.stop())
    cameraStream.value = null
  }
}

function validate() {
  errors.fullName = form.fullName ? '' : 'Full name is required.'
  errors.contactNumber = form.contactNumber ? '' : 'Contact number is required.'
  errors.gender = form.gender ? '' : 'Gender is required.'
  errors.address = form.address ? '' : 'Address is required.'
  errors.pin = form.pin ? '' : 'Pin Code is required.'
  errors.face = form.faceFile ? '' : 'Face photo is required.'
  errors.aadhar = form.aadharFile ? '' : 'Aadhaar photo is required.'
  errors.docType = form.docType ? '' : 'Document type is required.'
  errors.aadhaarNumber = form.aadhaarNumber ? '' : 'Aadhaar number is required.'

  return !Object.values(errors).some(v => v)
}

async function onSubmit() {
  if (!validate()) return

  const fd = new FormData()
  fd.append('full_name', form.fullName)
  fd.append('contact_number', form.contactNumber)
  fd.append('gender', form.gender)
  fd.append('emergency_contact', form.emergencyNumber || '')
  fd.append('home_town_address', form.address)
  fd.append('pin_code', form.pin)
  fd.append('vehicle_number', form.vehicle || '')
  if (form.faceFile) fd.append('face_photo', form.faceFile)
  if (form.aadharFile) fd.append('aadhaar_photo', form.aadharFile)
  fd.append('document_type', form.docType)
  fd.append('aadhaar_number', form.aadhaarNumber)
  fd.append('is_submitted', 'true')

  try {
    let res = null
    for (const base of apiBases) {
      try {
        const url = `${base}/register`
        res = await fetch(url, { method: 'POST', body: fd })
        if (!res.ok) {
          const err = await res.json().catch(() => ({ detail: res.statusText || 'Unknown error' }))
          alert(`Submission failed: ${err.detail || res.statusText}`)
          return
        }
        break
      } catch (err) {
        console.warn('Submission attempt failed for base', base, err)
        res = null
        // try next base
      }
    }

    if (!res) {
      alert('Submission failed: could not reach the server. Ensure backend is running or configure proxy.')
      return
    }

    const saved = await res.json()
    alert('Registration successful!')
    // keep the saved visitor for preview
    lastSavedVisitor.value = saved
    // reset form
    form.fullName = ''
    form.contactNumber = ''
    form.gender = ''
    form.emergencyNumber = ''
    form.address = ''
    form.pin = ''
    form.vehicle = ''
    form.faceFile = null
    form.aadharFile = null
    form.docType = ''
    form.aadhaarNumber = ''
    form.confirm = false
    console.log('Saved visitor:', saved)
  } catch (err) {
    console.error('Submission error:', err)
    alert('An error occurred while submitting. Please try again.')
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.form-page {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding: 24px 16px;
  /* background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%); */
  min-height: 100vh;
}

.card {
  width: 100%;
  max-width: 420px;
  background: #ffffff;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  padding: 24px;
  font-family: system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
}

.card-top {
  text-align: center;
  margin-bottom: 20px;
}

.brand {
  width: 70px;
  height: 70px;
  object-fit: contain;
  margin-bottom: 12px;
}

.subtle {
  color: #0d7fb3;
  margin: 0 0 8px;
  font-weight: 600;
  font-size: 16px;
  letter-spacing: 0.5px;
}

.title {
  text-align: center;
  font-size: 22px;
  font-weight: 700;
  margin: 12px 0 20px;
  text-decoration: underline;
  color: #1a1a1a;
  text-decoration-color: #0d7fb3;
  text-underline-offset: 6px;
}

.field {
  margin-bottom: 14px;
}

.field label {
  display: block;
  font-weight: 600;
  margin-bottom: 8px;
  color: #2c3e50;
  font-size: 14px;
}

.req {
  color: #e74c3c;
  font-weight: 700;
}

.two-col {
  display: flex;
  gap: 12px;
}

.two-col .field {
  flex: 1;
  margin-bottom: 0;
}

input[type="text"],
input[type="tel"],
input[type="email"],
select,
textarea {
  width: 100%;
  padding: 11px 14px;
  border: 1.5px solid #bdc3c7;
  border-radius: 6px;
  font-size: 14px;
  font-family: inherit;
  transition: all 0.3s ease;
  background-color: #fafafa;
}

input[type="text"]:focus,
input[type="tel"]:focus,
input[type="email"]:focus,
select:focus,
textarea:focus {
  outline: none;
  border-color: #0d7fb3;
  background-color: #fff;
  box-shadow: 0 0 0 3px rgba(13, 127, 179, 0.1);
}

textarea {
  resize: vertical;
  min-height: 80px;
}

.file-input-group {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
}

.hidden-input {
  display: none;
}

.file-btn,
.camera-btn {
  flex: 1;
  min-width: 120px;
  padding: 10px 14px;
  border: 1.5px solid #0d7fb3;
  background: #e8f4f8;
  color: #0d7fb3;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  font-size: 13px;
  transition: all 0.3s ease;
}

.file-btn:hover,
.camera-btn:hover {
  background: #0d7fb3;
  color: white;
  transform: translateY(-2px);
}

.file-name {
  display: block;
  font-size: 12px;
  color: #27ae60;
  margin-top: 4px;
  font-weight: 500;
}

.hint {
  display: block;
  color: #7f8c8d;
  font-size: 12px;
  margin-top: 6px;
}

.checkbox-row label {
  display: flex;
  gap: 8px;
  align-items: center;
  color: #2c3e50;
  font-weight: 500;
}

.checkbox-row input[type="checkbox"] {
  cursor: pointer;
  width: 18px;
  height: 18px;
}

.actions {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.btn {
  background: linear-gradient(135deg, #0288d1 0%, #0077c8 100%);
  color: white;
  border: none;
  padding: 12px 32px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  box-shadow: 0 4px 12px rgba(2, 120, 200, 0.3);
  transition: all 0.3s ease;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(2, 120, 200, 0.4);
}

.btn:active {
  transform: translateY(0);
}

.error {
  color: #c62828;
  font-size: 12px;
  display: block;
  margin-top: 4px;
  font-weight: 500;
}

/* Camera Modal Styles */
.camera-modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.camera-modal-content {
  background: white;
  border-radius: 12px;
  padding: 20px;
  max-width: 500px;
  width: 90%;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.3);
}

.camera-modal-content h3 {
  text-align: center;
  margin-bottom: 16px;
  color: #2c3e50;
}

#camera-video {
  width: 100%;
  height: auto;
  border-radius: 8px;
  background: #000;
  margin-bottom: 16px;
}

.camera-modal-buttons {
  display: flex;
  gap: 12px;
}

.btn-secondary {
  flex: 1;
  padding: 10px 16px;
  background: #95a5a6;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-secondary:hover {
  background: #7f8c8d;
}

.btn-primary {
  flex: 1;
  padding: 10px 16px;
  background: linear-gradient(135deg, #27ae60 0%, #229954 100%);
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(39, 174, 96, 0.3);
}

@media (max-width: 480px) {
  .card {
    padding: 18px;
  }

  .file-btn,
  .camera-btn {
    min-width: 100px;
  }

  .title {
    font-size: 20px;
  }
}

/* Saved visitor preview */
.saved-section {
  margin-top: 20px;
  padding-top: 12px;
  border-top: 1px dashed #e0e0e0;
}
.saved-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px 18px;
  margin-top: 8px;
}
.img-col {
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.select-row {
  display: flex;
  gap: 10px;
  align-items: center;
}
.btn-small {
  padding: 8px 10px;
  min-width: 72px;
  font-size: 13px;
}
.thumb {
  max-width: 140px;
  border-radius: 8px;
  border: 1px solid #ddd;
}

@media (max-width: 480px) {
  .saved-grid {
    grid-template-columns: 1fr;
  }
}
</style>
