<template>
  <div class="d-flex"  :class="[isPotrait ? 'flex-column' : 'flex-row']">
    <canvas id="canvas" class="grey darken-4" :height="canvasSize" :width="canvasSize">
    </canvas>

    <div class="pa-10">
      <p class="title">
        Choose Color
      </p>

      <div class="d-flex flex-wrap">
        <v-btn
          v-for="color in colors" 
          :key="color" 
          class="mr-5 mb-5" 
          :color="color"
          @click="onNewColorSelected(color)"
        >
          <v-icon :color="color === selectedColor? 'white' : color">
            mdi-check
          </v-icon>
        </v-btn>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue, } from 'vue-property-decorator'
import CoordinateModel from '../model/coordinate_model'
import io from 'socket.io-client'

@Component
export default class DrawUI extends Vue {
  isMouseDown = false
  isPotrait = false
  canvas!: HTMLCanvasElement
  context!: CanvasRenderingContext2D
  sharedCoordinate: Record<string, CoordinateModel> = {}
  canvasSize = window.innerWidth / window.innerHeight < 1.2 ? window.innerWidth : window.innerHeight
  localCoordinate!: CoordinateModel
  selectedColor = '#EE44AA'
  socket = io(process.env.SERVER || '192.168.0.153:3000')
  colors: string[] = ['#3F51B5', '#2196F3','#00BCD4','#4CAF50','#FFC107', '#F44336', '#E91E63', '#EE44AA']

  mounted () {
    this.canvas = document.getElementById('canvas') as HTMLCanvasElement
    this.context = this.canvas.getContext('2d') as CanvasRenderingContext2D
    window.addEventListener('resize', () => {
      this.onResize()
    });
    this.onResize()
    this.setupSocket()
    this.setupHtmlEventListener()
  }

  onNewColorSelected (newColor: string) {
    this.selectedColor = newColor
  }

  onResize () {
    const ratio = window.innerWidth / window.innerHeight
    if (ratio < 1.2) {
      this.canvasSize = window.innerWidth
      this.isPotrait = true
    } else {
      this.canvasSize = window.innerHeight
      this.isPotrait = false
    }

  }

  setupSocket () {
    this.socket.emit('join', { room: this.$route.params.room, name: this.$route.params.username }, (err: Error) => {
      if (err) {
        alert(err)
      } 
    })

    this.socket.on('newCanvasData', (e: any) => {
      const x = e.x * this.canvas.width
      const y = e.y * this.canvas.height
      const coordinate = new CoordinateModel(x, y)
      if (!e.isInitialDot) {
        this.drawToCanvas(coordinate, this.sharedCoordinate[e.id], e.color)
      } 
      this.sharedCoordinate[e.id] = coordinate
    })
  }

  setupHtmlEventListener () {
    this.canvas.ontouchstart = this.onTouchStart
    this.canvas.ontouchmove = this.onTouchMove
    this.canvas.ontouchend = this.onTouchEnd
    this.canvas.onmousedown = this.onMouseDown
    this.canvas.onmousemove = this.onMouseMove
    this.canvas.onmouseup = this.onMouseUp
    this.canvas.onmouseleave = () => {
      this.isMouseDown = false
    }
  }

  emitCanvasData (coordinate: CoordinateModel, isInitialDot: boolean) {
    const x = (coordinate.x - this.canvas.offsetLeft) / this.canvas.width
    const y = (coordinate.y - this.canvas.offsetTop) / this.canvas.width
    this.socket.emit('createCanvasData',
      {
        color: this.selectedColor,
        x: x,
        y: y,
        isInitialDot: isInitialDot
      },
      (err: Error) => {
        if (err) {
          console.log('emit error ' + err)
          alert(err)
        } 
      }
    )
  }

  onTouchStart (e: TouchEvent) {
    e.preventDefault()
    const x = e.touches[0].clientX - this.canvas.offsetLeft
    const y = e.touches[0].pageY - this.canvas.offsetTop
    const coordinate = new CoordinateModel(x, y)
    this.drawPreprocessing(coordinate, true)
    this.isMouseDown = true
  }

  onTouchMove (e: TouchEvent) {
    e.preventDefault()
    if (this.isMouseDown) {
      const x = e.touches[0].clientX - this.canvas.offsetLeft
      const y = e.touches[0].pageY - this.canvas.offsetTop
      const coordinate = new CoordinateModel(x, y)
      this.drawPreprocessing(coordinate, false)
    }
  }

  onTouchEnd (e: TouchEvent) {
    e.preventDefault()
    this.isMouseDown = false
  }

  onMouseDown (e: MouseEvent) {
    e.preventDefault()
    const x = e.clientX - this.canvas.offsetLeft
    const y = e.pageY - this.canvas.offsetTop
    const coordinate = new CoordinateModel(x, y)
    this.drawPreprocessing(coordinate, true)
    this.isMouseDown = true
  }

  onMouseMove (e: MouseEvent) {
    e.preventDefault()
    if (this.isMouseDown) {
      const x = e.clientX - this.canvas.offsetLeft
      const y = e.pageY - this.canvas.offsetTop
      const coordinate = new CoordinateModel(x, y)
      this.drawPreprocessing(coordinate, false)
    }
  }

  onMouseUp (e: MouseEvent) {
    e.preventDefault()
    this.isMouseDown = false
  }

  drawPreprocessing (coordinate: CoordinateModel, isInitialDot: boolean) {
    if (isInitialDot) {
      this.localCoordinate = coordinate
      this.emitCanvasData(coordinate, true)
    } else {
      this.drawToCanvas(coordinate, this.localCoordinate, this.selectedColor)
      this.emitCanvasData(coordinate, false)
      this.localCoordinate = coordinate
    }
  }

  drawToCanvas (value: CoordinateModel, oldValue: CoordinateModel, color: string) {
    this.context.beginPath()
    this.context.moveTo(oldValue.x, oldValue.y)
    this.context.lineTo(value.x, value.y)
    this.context.strokeStyle = color
    this.context.lineWidth = 1
    this.context.stroke()
    this.context.closePath()
  }

  beforeMount() {
    this.socket.emit('disconnect', (err: Error) => {
      if (err) {
        alert(err)
      } 
    })
  }
}

</script>
