<template>
  <div>
    <div class="wrap">
      <div class="painter-header">
        <section class="line-shape">
          <span>形状</span>
          <ul>
            <li v-for="(item, index) in shapes" 
                :key="item" 
                @click="setShape(item)">{{item}}</li>
          </ul>
          </section>
        <section class="line-width">画笔大小</section>
        <section class="line-color">
          <span>画笔颜色</span>
          <ul>
            <li v-for="(item, index) in colors" 
                :key="item" 
                :class="{active: config.lineColor === item}"
                :style="{backgroundColor: item}" 
                @click="setColor(item)"></li>
          </ul>
          </section>
        <section class="line-width">画笔大小</section>
        <section class="painter-control">
          <span>操作</span>
          <ul>
            <li v-for="(control,index) in controls" 
                :key="index"
                @click="canvasControl(control.action, $event)"
                :title="control.title">{{control.title}}</li>
          </ul>
          </section>
        <section class="pic-generator">生成图像</section>
      </div>
      <div class="canvas-wrap">
        <canvas 
        id="canvas" 
        width="400" 
        height="400" 
        @mousedown="canvasDown" 
        @mousemove="canvasMove" 
        @mouseup="canvasUp"
        @touchstart="canvasDown" 
        @touchmove="canvasMove" 
        @touchend="canvasUp"></canvas>
        <canvas 
        id="canvas-track" 
        width="400" 
        height="400" 
        @mousedown="canvasDown" 
        @mousemove="canvasMove" 
        @mouseup="canvasUp"
        @touchstart="canvasDown" 
        @touchmove="canvasMove" 
        @touchend="canvasUp"></canvas>
    </div>

      </div>
  </div>
</template>

<script>
export default {
  name: 'painter',
  data () {
    return {
      canvas: null,
      canvasTrack: null,
      shapes: ['curve', 'rect'],
      colors: ['black', 'red', 'pink', 'blue', 'tomato'],
      canvasMouseMove: false,
      context: {},
      trackContext: {},
      config: {
        lineShape: 'curve',
        lineWidth: 1,
        lineColor: 'red'
      },
      controls: [
        {
          title: '上一步',
          action: 'prev'
        },
        {
          title: '下一步',
          action: 'next'
        },
        {
          title: '清空',
          action: 'clear'
        }
      ],
      shapeStart: {
        x: 0,
        y: 0
      }
    }
  },
  methods: {
    setCanvasStyle () {
      let {lineWidth, lineColor, lineShape} = this.config
      this.context.lineWidth = lineWidth
      this.context.lineColor = lineColor
      this.context.strokeStyle = lineColor
      
      // track
      this.trackContext.lineWidth = lineWidth
      this.trackContext.lineColor = lineColor
      this.trackContext.strokeStyle = lineColor
    },
    setShape (shape) {
      this.config.lineShape = shape
    },
    setColor (color) {
      this.config.lineColor = color
    },
    // 设置画笔大小
    setBrush () {
    },
    canvasDown (e) {
      this.setCanvasStyle()
      this.drawStrategies(this.config.lineShape).canvasDown.call(this, e)
    },
    canvasUp (e) {
      this.drawStrategies(this.config.lineShape).canvasUp.call(this, e)
    },
    canvasMove (e) {
      this.drawStrategies(this.config.lineShape).canvasMove.call(this, e)
    },
    canvasControl (action, e) {
      switch(action) {
        case 'prev':
        case 'next':
        case 'clear':
          this.clear()
          break
      }
    },
    getMousePos (e, canvas) {
      let rect = canvas.getBoundingClientRect()
      // canvasX = e.clientX - e.target.offsetLeft
      // canvasY = e.clientY - e.target.offsetTop
      return {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top
      }

    },
    clearTrack () {
      this.trackContext.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
    },
    clear () {
      this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
    },
    drawRect (e, context, clearFlag) {
      let endX, endY, w, h, width, height, offsetX, offsetY
      endX = this.getMousePos(e, this.canvas).x
      endY = this.getMousePos(e, this.canvas).y
      w = endX - this.shapeStart.x
      h = endY - this.shapeStart.y
      offsetX = (w < 0) ? w : 0
      offsetY = (h < 0) ? h : 0
      width = Math.abs(endX - this.shapeStart.x)
      height = Math.abs(endY - this.shapeStart.y)
      if (clearFlag) {
        context.clearRect(0, 0, this.canvas.width, this.canvas.height)
      }
      context.beginPath()
      context.rect(this.shapeStart.x + offsetX, this.shapeStart.y + offsetY, width, height)
      context.stroke()

    },
    /**
     * @param {string} type 'curve' or 'rect' and etc.
     */
    drawStrategies (type) {
      switch(type) {
        case 'curve':
          return {
            canvasDown (e) {
              console.log('canvasDown')
              this.canvasMouseMove = true
              let canvasX, canvasY
              canvasX = this.getMousePos(e, this.canvas).x
              canvasY = this.getMousePos(e, this.canvas).y
              console.log(canvasX, canvasY)
              this.context.beginPath()
              this.context.moveTo(canvasX, canvasY)
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                console.log('canvasMove')
                let canvasX, canvasY
                canvasX = this.getMousePos(e, this.canvas).x
                canvasY = this.getMousePos(e, this.canvas).y
                this.context.lineTo(canvasX, canvasY)
                this.context.stroke()
              }
            },
            canvasUp (e) {
              this.canvasMouseMove = false
            }
          }
        case 'rect':
          return {
            canvasDown (e) {
              this.canvasMouseMove = true
              let canvasX, canvasY
              this.shapeStart.x = this.getMousePos(e, this.canvas).x
              this.shapeStart.y = this.getMousePos(e, this.canvas).y
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                this.drawRect(e, this.trackContext, true)
              }
            },
            canvasUp (e) {
              this.canvasMouseMove = false
              this.clearTrack()
              this.drawRect(e, this.context, false)
            }
          }
        case 'circle':
        case 'line':
      }
    }
  },
  mounted () {
    this.canvas = document.getElementById('canvas')
    this.canvasTrack = document.getElementById('canvas-track')
    this.context = this.canvas.getContext('2d')
    this.trackContext = this.canvasTrack.getContext('2d')
    this.setCanvasStyle()
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
.painter-header {
  border: 1px solid black;
  width: 400px;
}
.painter-header li {
  cursor: pointer;
}
.line-color li {
  display: inline-block;
  width: 13px;
  height: 13px;
  margin: 0 3px;
}
.line-color .active {
  border: 2px solid black;
}

/* canvas */
canvas {
  border: 1px solid black;
  cursor: crosshair;
  position: absolute;
}
#canvas {
  z-index: 2;
}
#canvas-track {
  z-index: 3;
}
</style>
