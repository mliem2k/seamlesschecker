<template>
  <div class="app">
    <div
      class="drop-zone"
      :class="{ 'drop-zone-dragging': isDragging }"
      @dragover.prevent="handleDragOver"
      @dragleave="handleDragLeave"
      @drop.prevent="handleDrop"
      @wheel="handleWheel"
    >
      <div class="grid-overlay" v-show="showGrid" :style="gridOverlayStyle" aria-hidden="true"></div>

      <div class="backdrop">
        <div class="container" :class="{ 'container-hover': isDragging }">
          <div class="content">
            <div class="header-row">
              <transition name="bounce" appear>
                <h1 class="animated-title">Seamless Texture Checker by Claudio</h1>
              </transition>
              <div class="file-input-wrapper">
                <input
                  ref="fileInput"
                  type="file"
                  id="fileInput"
                  accept="image/*"
                  @change="handleFileInput"
                  class="file-input visually-hidden"
                  aria-label="Choose texture image file"
                />
                <label for="fileInput" class="file-input-label" title="Choose texture image">
                  <span class="icon" aria-hidden="true">📁</span>
                </label>
              </div>
            </div>

            <div class="slider-row">
              <div class="slider-container">
                <input
                  id="textureSize"
                  type="range"
                  v-model.number="textureSize"
                  min="1"
                  max="20"
                  step="0.1"
                  aria-label="Tile size"
                  :aria-valuetext="`${tilePx} pixels`"
                  class="slider"
                  :style="sliderFillStyle"
                />
              </div>
              <span class="size-value">{{ tilePx }}px</span>
            </div>

            <div class="toggle-row">
              <button
                type="button"
                class="toggle-btn"
                :class="{ 'toggle-btn-active': offsetEnabled }"
                :aria-pressed="offsetEnabled"
                @click="offsetEnabled = !offsetEnabled"
                title="Shift the tile by half a repeat to reveal seams sitting at the tile edge"
              >
                <span aria-hidden="true">⇄</span> Offset
              </button>
              <button
                type="button"
                class="toggle-btn"
                :class="{ 'toggle-btn-active': showGrid }"
                :aria-pressed="showGrid"
                @click="showGrid = !showGrid"
                title="Overlay lines at every tile boundary"
              >
                <span aria-hidden="true">▦</span> Grid
              </button>
              <label
                class="color-swatch"
                title="Set a test background color, useful for checking transparent textures. Double-click to copy the hex code."
              >
                <input
                  type="color"
                  v-model="bgColor"
                  @input="applyBgColor"
                  @dblclick.prevent="copyBgColorHex"
                  aria-label="Test background color"
                />
              </label>
              <button
                type="button"
                class="toggle-btn"
                :disabled="!canDownload"
                @click="downloadTiledImage"
                title="Download the current tiled pattern as a PNG"
              >
                <span aria-hidden="true">⬇</span> Download
              </button>
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
      isDragging: false,
      offsetEnabled: false,
      showGrid: false,
      currentObjectUrl: null,
      isRemoteImage: false,
      bgColor: '#ffffff'
    }
  },
  computed: {
    // Single source of truth for the tile size in pixels, shared by the
    // background-size, the grid overlay, and the on-screen readout.
    tilePx() {
      return Math.round(this.textureSize * 50)
    },
    backgroundPositionValue() {
      return this.offsetEnabled ? `${this.tilePx / 2}px ${this.tilePx / 2}px` : '0px 0px'
    },
    gridOverlayStyle() {
      return {
        backgroundSize: `${this.tilePx}px ${this.tilePx}px`,
        backgroundPosition: this.backgroundPositionValue
      }
    },
    sliderFillStyle() {
      const percent = ((this.textureSize - 1) / (20 - 1)) * 100
      return { '--fill': `${percent}%` }
    },
    // Exporting to canvas requires drawImage() on the loaded image, which
    // throws for cross-origin images without CORS headers (e.g. a pasted
    // remote URL). Only same-origin blob URLs (file/drag/paste-as-file) are
    // guaranteed exportable.
    canDownload() {
      return !!this.currentObjectUrl && !this.isRemoteImage
    }
  },
  watch: {
    tilePx() {
      this.applyBackgroundSizing()
    },
    offsetEnabled() {
      this.applyBackgroundSizing()
    }
  },
  methods: {
    handleFileSelect(files) {
      const file = files && files[0]
      if (!file) return

      // A blob object URL is a lightweight reference to the file's existing
      // bytes; no base64 re-encoding or extra decode pass, unlike
      // FileReader.readAsDataURL(). Revoke the previous one so we don't leak
      // it every time a new texture is loaded.
      const objectUrl = URL.createObjectURL(file)
      if (this.currentObjectUrl) {
        URL.revokeObjectURL(this.currentObjectUrl)
      }
      this.currentObjectUrl = objectUrl
      this.isRemoteImage = false

      document.body.style.backgroundImage = `url('${objectUrl}')`
      this.applyBackgroundSizing()
    },
    // A pasted image URL/link loads directly as a CSS background, same as
    // the reference tool's "paste a copied image link" support. It can't be
    // exported later (see canDownload) since it's cross-origin.
    loadRemoteImage(url) {
      if (this.currentObjectUrl) {
        URL.revokeObjectURL(this.currentObjectUrl)
      }
      this.currentObjectUrl = url
      this.isRemoteImage = true

      document.body.style.backgroundImage = `url('${url}')`
      this.applyBackgroundSizing()
    },
    applyBackgroundSizing() {
      document.body.style.backgroundSize = `${this.tilePx}px`
      document.body.style.backgroundPosition = this.backgroundPositionValue
    },
    applyBgColor() {
      document.body.style.backgroundColor = this.bgColor
    },
    async copyBgColorHex() {
      try {
        await navigator.clipboard.writeText(this.bgColor)
      } catch (e) {
        console.error('Clipboard write failed:', e)
      }
    },
    handleFileInput(event) {
      this.handleFileSelect(event.target.files)
      event.target.value = '' // Reset input
    },
    handleDragOver() {
      this.isDragging = true
    },
    handleDragLeave() {
      this.isDragging = false
    },
    handleDrop(event) {
      this.handleDragLeave()
      this.handleFileSelect(event.dataTransfer.files)
    },
    // Shift+scroll zoom, matching the reference tool's shortcut. Plain
    // scroll is left alone (there's nothing to scroll; the page is fixed).
    handleWheel(event) {
      if (!event.shiftKey) return
      event.preventDefault()
      const direction = event.deltaY > 0 ? -1 : 1
      const next = this.textureSize + direction * 0.5
      this.textureSize = Math.min(20, Math.max(1, next))
    },
    handlePaste(event) {
      const items = event.clipboardData.items
      for (let i = 0; i < items.length; i++) {
        if (items[i].type.startsWith('image/')) {
          const file = items[i].getAsFile()
          this.handleFileSelect([file])
          return
        }
      }

      // No image file on the clipboard, fall back to a pasted image URL/link.
      const text = event.clipboardData.getData('text/plain')?.trim()
      if (text && /^https?:\/\/\S+$/i.test(text)) {
        this.loadRemoteImage(text)
      }
    },
    async downloadTiledImage() {
      if (!this.canDownload) return

      const img = new Image()
      img.src = this.currentObjectUrl
      await img.decode()

      const tile = this.tilePx
      const tileCount = Math.max(2, Math.ceil(1200 / tile))
      const canvasSize = tile * tileCount
      const canvas = document.createElement('canvas')
      canvas.width = canvasSize
      canvas.height = canvasSize
      const ctx = canvas.getContext('2d')

      ctx.fillStyle = this.bgColor
      ctx.fillRect(0, 0, canvasSize, canvasSize)

      const offset = this.offsetEnabled ? tile / 2 : 0
      for (let y = -offset; y < canvasSize; y += tile) {
        for (let x = -offset; x < canvasSize; x += tile) {
          ctx.drawImage(img, x, y, tile, tile)
        }
      }

      canvas.toBlob((blob) => {
        if (!blob) return
        const url = URL.createObjectURL(blob)
        const a = document.createElement('a')
        a.href = url
        a.download = 'tiled-texture.png'
        a.click()
        URL.revokeObjectURL(url)
      })
    }
  },
  mounted() {
    this.applyBackgroundSizing()
    document.addEventListener('paste', this.handlePaste)
  },
  beforeUnmount() {
    document.removeEventListener('paste', this.handlePaste)
    if (this.currentObjectUrl) {
      URL.revokeObjectURL(this.currentObjectUrl)
    }
  }
}
</script>

<style>
:root {
  --color-primary: #007bff;
  --color-primary-deep: #0056b3;
  --color-ink: #333333;
  --color-white-panel: rgba(255, 255, 255, 0.95);
  --color-page-bg: #f0f0f0;
  --color-track-bg: #ddd;
  --color-scrim-idle: rgba(0, 0, 0, 0.1);
  --color-scrim-drag: rgba(0, 0, 0, 0.2);
  --color-inspection-guide: rgba(255, 0, 0, 0.35);
  --radius-sm: 5px;
  --radius-lg: 15px;
  --space-xs: 6px;
  --space-sm: 10px;
  --space-md: 15px;
  --space-lg: 20px;
}

html, body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100vh;
  background-size: 100px;
  font-family: "Georgia", "Calibri";
  background-image: url("/default.svg");
  background-color: var(--color-page-bg);
  overflow: hidden;
}

.app {
  width: 100%;
  height: 100%;
}

.visually-hidden {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border: 0;
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
  background-color: var(--color-scrim-idle);
  color: var(--color-primary);
  transition: all 0.3s ease;
}

.drop-zone-dragging {
  background-color: var(--color-scrim-drag);
  color: var(--color-ink);
}

.grid-overlay {
  position: fixed;
  inset: 0;
  z-index: 1200;
  pointer-events: none;
  background-image:
    linear-gradient(to right, rgba(255, 0, 0, 0.35) 1px, transparent 1px),
    linear-gradient(to bottom, rgba(255, 0, 0, 0.35) 1px, transparent 1px);
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
  width: min(400px, calc(100vw - 2rem));
  background-color: var(--color-white-panel);
  padding: var(--space-lg);
  border-radius: var(--radius-lg);
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.container::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: var(--radius-lg);
  animation: float 6s ease-in-out infinite;
  pointer-events: none;
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
  gap: var(--space-sm);
  margin-bottom: var(--space-md);
}

h1 {
  font-size: 1.1em;
  margin: 0;
  color: var(--color-ink);
  font-family: 'Comic Sans MS', 'Comic Sans', cursive;
  white-space: nowrap;
  flex: 1;
}

.file-input-wrapper {
  margin: 0;
}

.file-input-label {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  min-width: 44px;
  min-height: 44px;
  padding: var(--space-xs) var(--space-sm);
  background: var(--color-primary);
  color: white;
  border-radius: var(--radius-lg);
  cursor: pointer;
  transition: all 0.3s ease;
  font-size: 0.8em;
  white-space: nowrap;
  animation: pulse 2s infinite;
}

.file-input-label:hover {
  background: var(--color-primary-deep);
  transform: translateY(-2px);
}

.file-input:focus-visible + .file-input-label,
.file-input:focus + .file-input-label {
  outline: 2px solid var(--color-ink);
  outline-offset: 2px;
}

.slider-row {
  display: flex;
  align-items: center;
  gap: var(--space-sm);
  margin: var(--space-md) 0;
}

.slider-container {
  flex: 1;
  padding: 0 var(--space-xs);
}

.slider {
  -webkit-appearance: none;
  width: 100%;
  height: 8px;
  background: var(--color-track-bg);
  border-radius: var(--radius-sm);
  outline: none;
  cursor: pointer;
  margin: 8px 0;
  transition: all 0.2s ease;
  position: relative;
  overflow: visible;
}

.slider::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: var(--fill, 0%);
  background: linear-gradient(90deg,
    #ff0000, #ff7f00, #ffff00, #00ff00,
    #0000ff, #4b0082, #8b00ff, #ff0000);
  background-size: 200% 100%;
  animation: rainbow-track 3s linear infinite;
  border-radius: var(--radius-sm);
  z-index: 1;
}

.slider::-webkit-slider-runnable-track {
  background: transparent;
}

.slider::-moz-range-track {
  background: transparent;
}

.slider::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 20px;
  height: 20px;
  background: var(--color-primary);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  position: relative;
  z-index: 2;
}

.slider::-webkit-slider-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(0, 123, 255, 0.4);
}

.slider::-moz-range-thumb {
  width: 20px;
  height: 20px;
  background: var(--color-primary);
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  position: relative;
  z-index: 2;
}

.slider::-moz-range-thumb:hover {
  transform: scale(1.1);
  box-shadow: 0 4px 8px rgba(0, 123, 255, 0.4);
}

.size-value {
  display: inline-block;
  font-size: 0.85em;
  color: var(--color-ink);
  min-width: 3.5em;
  text-align: right;
}

.toggle-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: var(--space-sm);
}

.toggle-btn {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  min-height: 36px;
  padding: 6px 14px;
  background: white;
  color: var(--color-ink);
  border: 1px solid var(--color-track-bg);
  border-radius: var(--radius-lg);
  cursor: pointer;
  font-size: 0.85em;
  transition: all 0.2s ease;
}

.toggle-btn:hover {
  border-color: var(--color-primary);
}

.toggle-btn:focus-visible {
  outline: 2px solid var(--color-ink);
  outline-offset: 2px;
}

.toggle-btn-active {
  background: var(--color-primary);
  color: white;
  border-color: var(--color-primary);
}

.toggle-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.toggle-btn:disabled:hover {
  border-color: var(--color-track-bg);
}

.color-swatch {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  cursor: pointer;
}

.color-swatch input[type='color'] {
  width: 36px;
  height: 36px;
  padding: 0;
  border: 1px solid var(--color-track-bg);
  border-radius: 50%;
  cursor: pointer;
}

.color-swatch input[type='color']::-webkit-color-swatch-wrapper {
  padding: 0;
  border-radius: 50%;
  overflow: hidden;
}

.color-swatch input[type='color']::-webkit-color-swatch {
  border: none;
  border-radius: 50%;
}

.color-swatch input[type='color']::-moz-color-swatch {
  border: none;
  border-radius: 50%;
}

.color-swatch:has(input:focus-visible) {
  outline: 2px solid var(--color-ink);
  outline-offset: 2px;
}

/* Animation styles */
.bounce-enter-active {
  animation: bounce-in 0.8s;
}

.bounce-leave-active {
  animation: bounce-in 0.8s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
    opacity: 0;
  }
  50% {
    transform: scale(1.2);
  }
  100% {
    transform: scale(1);
    opacity: 1;
  }
}

.animated-title {
  animation: rainbow 5s linear infinite, float-title 3s ease-in-out infinite;
  background: linear-gradient(90deg, #ff0000, #ff7f00, #ffff00, #00ff00, #0000ff, #4b0082, #8b00ff);
  background-size: 400% 400%;
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  /* Keeps every gradient stop (yellow especially) legible against the
     near-white panel; the stroke sits outside the clipped fill so the
     rainbow itself is untouched. */
  -webkit-text-stroke: 0.5px rgba(0, 0, 0, 0.35);
  text-shadow: none;
  display: inline-block;
}

@keyframes rainbow {
  0% {
    background-position: 0% 50%;
  }
  100% {
    background-position: 100% 50%;
  }
}

@keyframes float {
  0% {
    transform: translateY(0px);
  }
  50% {
    transform: translateY(-10px);
  }
  100% {
    transform: translateY(0px);
  }
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.05);
  }
  100% {
    transform: scale(1);
  }
}

@keyframes float-title {
  0% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
  100% {
    transform: translateY(0);
  }
}

@keyframes rainbow-track {
  0% {
    background-position: 0% 50%;
  }
  100% {
    background-position: 200% 50%;
  }
}

@media (prefers-reduced-motion: reduce) {
  .container::before,
  .file-input-label,
  .animated-title,
  .slider::before {
    animation: none !important;
  }

  .bounce-enter-active,
  .bounce-leave-active {
    animation: none !important;
  }

  .drop-zone,
  .container,
  .file-input-label,
  .toggle-btn {
    transition: none !important;
  }
}
</style>
