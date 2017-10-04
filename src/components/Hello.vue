<template>
  <div>
    <div class="wrap">
      <div class="painter-header">
        <section class="line-color">
          <span>画笔颜色</span>
          <ul>
            <li v-for="(item, index) in colors" 
                :key="item" 
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
      width="300" 
      height="300" 
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
      colors: ['red', 'pink', 'blue', 'black', 'tomato'],
      canvasMouseMove: false,
      context: {},
      config: {
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
      ]
    }
  },
  methods: {
    setCanvasStyle () {
      let {lineWidth, lineColor} = this.config
      this.context.lineWidth = lineWidth
      this.context.lineColor = lineColor
    },
    setColor (color) {
      this.context.strokeStyle = color;
    },
    // 设置画笔大小
    setBrush () {
    },
    canvasDown (e) {
      console.log('canvasDown')
      this.canvasMouseMove = true
      const canvasX = e.clientX - e.target.offsetLeft
      const canvasY = e.clientY - e.target.offsetTop
      console.log(canvasX, canvasY)
      this.setCanvasStyle()
      this.context.beginPath()
      this.context.moveTo(canvasX, canvasY)
    },
    canvasUp (e) {
      this.canvasMouseMove = false
    },
    canvasMove (e) {
      if (this.canvasMouseMove) {
        console.log('canvasMove')
        let canvasX, canvasY
        canvasX = e.clientX - e.target.offsetLeft
        canvasY = e.clientY - e.target.offsetTop
        this.context.lineTo(canvasX, canvasY)
        this.context.stroke()

      }
    },
    canvasControl (action, e) {
      switch(action) {
        case 'prev':
        case 'next':
        case 'clear':
          this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
          break
      }
    }
  },
  mounted () {
    const canvas = document.getElementById('canvas')
    this.context = canvas.getContext('2d')
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
  width: 300px;
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
canvas {
  border: 1px solid black;
  cursor: crosshair;
}
</style>
