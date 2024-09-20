<template>
  <div class="whiteboard">
    <canvas id="canvas"></canvas>
    <ToolbarComponent
      @setTool="setTool"
      @setColor="setColor"
      @setBrushSize="setBrushSize"
      @clearCanvas="clearCanvas"
    />
  </div>
</template>

<script>
import * as fabric from 'fabric'
import io from 'socket.io-client'
import ToolbarComponent from './ToolbarComponent.vue'

export default {
  props: ['user'],
  components: {
    ToolbarComponent
  },
  data() {
    return {
      canvas: null,
      socket: null,
      currentTool: 'pencil',
      isDrawing: false,
      drawingObject: null,
      selectedColor: '#000000',
      brushSize: 2
    }
  },
  mounted() {
    this.setupCanvas()
    this.setupSocket()
    this.loadFromLocalStorage()
  },
  methods: {
    setupCanvas() {
      this.canvas = new fabric.Canvas('canvas', {
        width: window.innerWidth >= 1000 ? 800 : window.innerWidth - 100,
        height: window.innerWidth <= 770 ? window.innerHeight / 2 - 70 : 500,
        backgroundColor: '#fff'
      })

      this.setPencilTool()

      this.loadFromLocalStorage()

      this.canvas.on('path:created', this.sendDrawing)

      this.canvas.on('object:added', this.saveToLocalStorage)
      this.canvas.on('object:modified', this.saveToLocalStorage)

      this.canvas.on('mouse:down', this.onMouseDown)
      this.canvas.on('mouse:move', this.onMouseMove)
      this.canvas.on('mouse:up', this.onMouseUp)

      this.updateCursor()
    },
    saveToLocalStorage() {
      const canvasJSON = JSON.stringify(this.canvas.toDatalessJSON())
      localStorage.setItem(`whiteboardCanvas-${this.user.name}`, canvasJSON)
      console.log(this.user.name)
    },

    loadFromLocalStorage() {
      console.log(window.innerWidth, window.innerHeight)

      const savedCanvas = localStorage.getItem(`whiteboardCanvas-${this.user.name}`)
      if (savedCanvas) {
        this.canvas.loadFromJSON(savedCanvas, () => {
          this.canvas.renderAll()
        })
      }
    },

    saveCanvasManually() {
      this.saveToLocalStorage()
    },

    loadCanvasManually() {
      this.loadFromLocalStorage()
    },

    clearCanvas() {
      this.canvas.clear()
      this.canvas.setBackgroundColor('#fff')
      localStorage.removeItem('whiteboardCanvas')
    },

    setupSocket() {
      this.socket = io('https://foyr-be.onrender.com')

      this.socket.on('connect', () => {
        this.socket.emit('join', { user: this.user })
      })

      this.socket.on('drawing', (data) => {
        fabric.util.enlivenObjects([data], (objects) => {
          objects.forEach((obj) => {
            this.canvas.add(obj)
          })
        })
      })
    },

    setPencilTool() {
      this.canvas.isDrawingMode = true
      this.canvas.freeDrawingBrush = new fabric.PencilBrush(this.canvas)
      this.canvas.freeDrawingBrush.color = this.selectedColor
      this.canvas.freeDrawingBrush.width = this.brushSize
      this.updateCursor()
    },

    sendDrawing(event) {
      const drawing = event.path
      this.socket.emit('drawing', drawing)
    },

    setTool(tool) {
      this.currentTool = tool

      if (tool === 'pencil') {
        this.setPencilTool()
      } else if (tool === 'eraser') {
        this.canvas.isDrawingMode = true
        this.canvas.freeDrawingBrush.color = '#fff'
      } else {
        this.canvas.isDrawingMode = false
      }

      this.updateCursor()
    },

    updateCursor() {
      const canvasElement = this.canvas.upperCanvasEl

      if (this.currentTool === 'pencil') {
        canvasElement.classList.add('pencil-cursor')
        canvasElement.classList.remove('eraser-cursor')
      } else if (this.currentTool === 'eraser') {
        canvasElement.classList.add('eraser-cursor')
        canvasElement.classList.remove('pencil-cursor')
      } else {
        canvasElement.classList.remove('pencil-cursor', 'eraser-cursor')
      }
    },

    setColor(color) {
      this.selectedColor = color
      if (this.currentTool === 'pencil') {
        this.canvas.freeDrawingBrush.color = color
      }
    },

    setBrushSize(size) {
      this.brushSize = size
      if (this.currentTool === 'pencil' || this.currentTool === 'eraser') {
        this.canvas.freeDrawingBrush.width = size
      }
    },

    clearCanvas() {
      this.canvas.clear()
      this.canvas.setBackgroundColor('#fff')
    },

    onMouseDown(event) {
      const pointer = this.canvas.getPointer(event.e)

      if (this.currentTool === 'rectangle') {
        this.isDrawing = true
        this.drawingObject = new fabric.Rect({
          left: pointer.x,
          top: pointer.y,
          width: 0,
          height: 0,
          fill: 'transparent',
          stroke: this.selectedColor,
          strokeWidth: this.brushSize,
          originX: 'left',
          originY: 'top'
        })
        this.canvas.add(this.drawingObject)
      } else if (this.currentTool === 'circle') {
        this.isDrawing = true
        this.drawingObject = new fabric.Circle({
          left: pointer.x,
          top: pointer.y,
          radius: 1,
          fill: 'transparent',
          stroke: this.selectedColor,
          strokeWidth: this.brushSize
        })
        this.canvas.add(this.drawingObject)
      } else if (this.currentTool === 'line') {
        this.isDrawing = true
        this.drawingObject = new fabric.Line([pointer.x, pointer.y, pointer.x, pointer.y], {
          stroke: this.selectedColor,
          strokeWidth: this.brushSize
        })
        this.canvas.add(this.drawingObject)
      } else if (this.currentTool === 'eraser') {
        this.canvas.isDrawingMode = true
        this.canvas.freeDrawingBrush.color = '#fff'
      }
    },

    onMouseMove(event) {
      if (!this.isDrawing || !this.drawingObject) return

      const pointer = this.canvas.getPointer(event.e)

      if (this.currentTool === 'rectangle') {
        let left = Math.min(this.drawingObject.left, pointer.x)
        let top = Math.min(this.drawingObject.top, pointer.y)
        let width = Math.abs(pointer.x - this.drawingObject.left)
        let height = Math.abs(pointer.y - this.drawingObject.top)

        this.drawingObject.set({
          left,
          top,
          width,
          height
        })
      } else if (this.currentTool === 'circle') {
        const radius = Math.sqrt(
          Math.pow(pointer.x - this.drawingObject.left, 2) +
            Math.pow(pointer.y - this.drawingObject.top, 2)
        )
        this.drawingObject.set({ radius })
      } else if (this.currentTool === 'line') {
        this.drawingObject.set({
          x2: pointer.x,
          y2: pointer.y
        })
      }
      this.canvas.renderAll()
    },

    onMouseUp() {
      if (this.isDrawing) {
        this.isDrawing = false
        this.sendDrawing({ path: this.drawingObject.toObject() })
        this.drawingObject = null
      }
    }
  }
}
</script>

<style scoped>
.whiteboard {
  display: flex;
  flex-direction: column;
  align-items: center;
  /* height: 100vh; */
  overflow-y: scroll;
  overflow-x: hidden;
}
canvas {
  border: 1px solid #ddd;
}

.pencil-cursor {
  cursor: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" fill="%23000" viewBox="0 0 512 512" width="20" height="20"><path d="M497.9 142.1L369.9 14.1c-18.7-18.7-49.1-18.7-67.9 0L27.9 289.1c-5.9 5.9-9.4 13.8-9.9 22.2l-16 160c-1.4 13.6 3.7 27 13.7 37s23.4 15.1 37 13.7l160-16c8.4-.5 16.3-4 22.2-9.9l274-274c18.8-18.8 18.8-49.2 0-68zM124.1 312L312 124.1l76 76L200.1 388l-99.9 11.1 11.1-99.9zm247.8-18.5L218.5 446.9c-.8.8-1.9 1.2-3 1.1l-85.2-8.5 83.4-83.4 160.5-160.5 76 76-79.3 79.3z"/></svg>'),
    auto;
}

.eraser-cursor {
  cursor: url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 512 512" fill="%23000000" width="20" height="20"><path d="M497.9 142.1L369.9 14.1c-18.7-18.7-49.1-18.7-67.9 0L27.9 289.1c-5.9 5.9-9.4 13.8-9.9 22.2l-16 160c-1.4 13.6 3.7 27 13.7 37s23.4 15.1 37 13.7l160-16c8.4-.5 16.3-4 22.2-9.9l274-274c18.8-18.8 18.8-49.2 0-68zM124.1 312L312 124.1l76 76L200.1 388l-99.9 11.1 11.1-99.9zm247.8-18.5L218.5 446.9c-.8.8-1.9 1.2-3 1.1l-85.2-8.5 83.4-83.4 160.5-160.5 76 76-79.3 79.3z"/></svg>'),
    auto;
}
</style>
