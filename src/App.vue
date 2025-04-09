<template>
  <div class="app">
    <div class="drop-zone" 
         @dragover.prevent="handleDragOver"
         @dragleave="handleDragLeave"
         @drop.prevent="handleDrop"
         :style="dropZoneStyle">
      <div class="backdrop">
        <div class="container" :class="{ 'container-hover': isDragging }">
          <div class="content">
            <div class="header-row">
              <transition name="flip" appear>
                <h1 class="animated-title">Seamless Texture Checker by Claudio</h1>
              </transition>
              <div class="file-input-wrapper">
                <input 
                  type="file" 
                  id="fileInput" 
                  accept="image/*" 
                  @change="handleFileInput"
                  class="file-input"
                />
                <label for="fileInput" class="file-input-label">
                  <span class="icon">📁</span>
                </label>
              </div>
            </div>
            <div class="slider-container">
              <input
                id="textureSize"
                type="range"
                v-model="textureSize"
                min="1"
                max="20"
                step="0.1"
                @input="updateTextureSize"
                class="slider"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      textureSize: 3,
      dropZoneStyle: {
        backgroundColor: 'rgba(0, 0, 0, 0.1)',
        color: '#007bff'
      },
      isDragging: false
    }
  },
  methods: {
    handleFileSelect(files) {
      if (!files || !files[0]) return
      const image = new Image()
      const reader = new FileReader()

      reader.onload = (e) => {
        image.src = e.target.result
        image.onload = () => {
          document.body.style.backgroundImage = `url('${e.target.result}')`
        }
      }
      reader.readAsDataURL(files[0])
    },
    handleFileInput(event) {
      this.handleFileSelect(event.target.files)
      event.target.value = '' // Reset input
    },
    handleDragOver() {
      this.isDragging = true
      this.dropZoneStyle = {
        backgroundColor: 'rgba(0, 0, 0, 0.2)',
        color: '#333'
      }
    },
    handleDragLeave() {
      this.isDragging = false
      this.dropZoneStyle = {
        backgroundColor: 'rgba(0, 0, 0, 0.1)',
        color: '#007bff'
      }
    },
    handleDrop(event) {
      this.handleDragLeave()
      this.handleFileSelect(event.dataTransfer.files)
    },
    updateTextureSize() {
      const widthpx = this.textureSize * 50
      document.body.style.backgroundSize = `${widthpx}px`
    }
  },
  mounted() {
    document.addEventListener('paste', (event) => {
      const items = event.clipboardData.items
      for (let i = 0; i < items.length; i++) {
        if (items[i].type.startsWith('image/')) {
          const file = items[i].getAsFile()
          this.handleFileSelect([file])
        }
      }
    })
  }
}
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100vh;
  background-size: 150px;
  font-family: "Georgia", "Calibri";
  background-image: url("default.jpg");
  background-color: #f0f0f0;
  overflow: hidden;
}

.app {
  width: 100%;
  height: 100%;
}

.drop-zone {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1500;
  transition: all 0.3s ease;
}

.backdrop {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 999;
  display: flex;
  justify-content: center;
  align-items: center;
}

.container {
  width: 400px;
  background-color: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 15px;
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  animation: float3d-container 8s ease-in-out infinite;
  transform-style: preserve-3d;
  perspective: 1000px;
}

.container-hover {
  transform: translateX(-50%) scale(1.02);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.15);
}

.content {
  text-align: center;
}

.header-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  margin-bottom: 15px;
}

h1 {
  font-size: 1.1em;
  margin: 0;
  color: #333;
  font-family: 'Comic Sans MS', 'Comic Sans', cursive;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
  white-space: nowrap;
  flex: 1;
}

.file-input-wrapper {
  margin: 0;
}

.file-input {
  display: none;
}

.file-input-label {
  display: inline-block;
  padding: 6px 12px;
  background: #007bff;
  color: white;
  border-radius: 15px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.8em;
  white-space: nowrap;
  animation: pulse3d 2s infinite;
  transform-style: preserve-3d;
}

.file-input-label:hover {
  background: #0056b3;
  transform: translateY(-2px);
}

.slider-container {
  margin: 15px 0;
  padding: 0 10px;
}

.slider {
  -webkit-appearance: none;
  width: 100%;
  height: 8px;
  background: #ddd;
  border-radius: 5px;
  outline: none;
  cursor: pointer;
  margin: 8px 0;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
}

.slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
}

.slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: #007bff;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.3s ease;
}

.slider::-moz-range-thumb:hover {
  transform: scale(1.1);
}

.size-value {
  display: inline-block;
  margin-top: 8px;
  font-size: 1em;
  color: #333;
}

/* Animation styles */
.flip-enter-active {
  animation: flip-in 1s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

.flip-leave-active {
  animation: flip-out 1s cubic-bezier(0.25, 0.46, 0.45, 0.94) both;
}

@keyframes flip-in {
  0% {
    transform: perspective(400px) rotateX(90deg) scale(0.3);
    opacity: 0;
  }
  100% {
    transform: perspective(400px) rotateX(0) scale(1);
    opacity: 1;
  }
}

@keyframes flip-out {
  0% {
    transform: perspective(400px) rotateX(0) scale(1);
    opacity: 1;
  }
  100% {
    transform: perspective(400px) rotateX(-90deg) scale(0.3);
    opacity: 0;
  }
}

.animated-title {
  animation: rainbow 5s linear infinite, float3d 6s ease-in-out infinite;
  background: linear-gradient(90deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #8b00ff);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  text-shadow: none;
  transform-style: preserve-3d;
  perspective: 1000px;
}

@keyframes rainbow {
  0% {
    background-position: 0% 50%;
  }
  100% {
    background-position: 100% 50%;
  }
}

@keyframes float3d {
  0% {
    transform: perspective(1000px) rotateX(0deg) rotateY(0deg) translateZ(0);
  }
  25% {
    transform: perspective(1000px) rotateX(5deg) rotateY(5deg) translateZ(20px);
  }
  50% {
    transform: perspective(1000px) rotateX(0deg) rotateY(0deg) translateZ(0);
  }
  75% {
    transform: perspective(1000px) rotateX(-5deg) rotateY(-5deg) translateZ(20px);
  }
  100% {
    transform: perspective(1000px) rotateX(0deg) rotateY(0deg) translateZ(0);
  }
}

.container {
  animation: float3d-container 8s ease-in-out infinite;
}

@keyframes float3d-container {
  0% {
    transform: perspective(1000px) translateX(-50%) translateY(0) rotateX(0deg) rotateY(0deg);
  }
  25% {
    transform: perspective(1000px) translateX(-50%) translateY(-10px) rotateX(2deg) rotateY(2deg);
  }
  50% {
    transform: perspective(1000px) translateX(-50%) translateY(0) rotateX(0deg) rotateY(0deg);
  }
  75% {
    transform: perspective(1000px) translateX(-50%) translateY(-10px) rotateX(-2deg) rotateY(-2deg);
  }
  100% {
    transform: perspective(1000px) translateX(-50%) translateY(0) rotateX(0deg) rotateY(0deg);
  }
}

.file-input-label {
  animation: pulse3d 2s infinite;
}

@keyframes pulse3d {
  0% {
    transform: perspective(1000px) scale(1) translateZ(0);
  }
  50% {
    transform: perspective(1000px) scale(1.1) translateZ(10px);
  }
  100% {
    transform: perspective(1000px) scale(1) translateZ(0);
  }
}
</style> 