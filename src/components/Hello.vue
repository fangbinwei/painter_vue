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
    </div>
  </div>
</template>

<script>
export default {
  name: 'painter',
  data () {
    return {
      canvas: null,
      shapes: ['curve', 'rect'],
      colors: ['black', 'red', 'pink', 'blue', 'tomato'],
      canvasMouseMove: false,
      context: {},
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
      this.context.strokeStyle = this.config.lineColor
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
          this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
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
    /**@argument type:'curve' */
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
                console.log(this.context)
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
                let endX, endY, w, h, width, height, offsetX, offsetY
                endX = this.getMousePos(e, this.canvas).x
                endY = this.getMousePos(e, this.canvas).y
                w = endX - this.shapeStart.x
                h = endY - this.shapeStart.y
                offsetX = (w < 0) ? w : 0
                offsetY = (h < 0) ? h : 0
                width = Math.abs(endX - this.shapeStart.x)
                height = Math.abs(endY - this.shapeStart.y)
                this.context.clearRect(0, 0, this.canvas.width, this.canvas.height)
                this.context.beginPath()
                this.context.rect(this.shapeStart.x + offsetX, this.shapeStart.y + offsetY, width, height)
                this.context.stroke()
              }
            },
            canvasUp (e) {
              this.canvasMouseMove = false
            }
          }
        case 'circle':
        case 'line':
      }
    }
  },
  mounted () {
    this.canvas = document.getElementById('canvas')
    this.context = this.canvas.getContext('2d')
    this.setCanvasStyle()
    // (function draw () {
    //   var canvas = document.getElementById('canvas')
    //   if (canvas.getContext) {
    //     var ctx = canvas.getContext('2d')
    //     ctx.beginPath()
    //     ctx.arc(100, 100, 99, 2 * Math.PI, false)

    //     ctx.moveTo(194, 100)
    //     ctx.arc(100, 100, 94, 0, 2* Math.PI, false)

    //     ctx.translate(100, 100)
    //     ctx.rotate(1)
    //     ctx.moveTo(0,0)
    //     ctx.lineTo(0, -85)
    //     ctx.stroke()
    //   }
    // })()
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
canvas {
  border: 1px solid black;
  cursor: crosshair;
}
</style>
