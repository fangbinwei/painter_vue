<template>
  <div class='hello'>
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
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      colors: ['red', 'tomato'],
      canvasMouseMove: false,
      context: {},
      config: {
        lineWidth: 1,
        lineColor: 'red'
      }
    }
  },
  methods: {
    setCanvasStyle () {
      let {lineWidth, lineColor} = this.config
      this.context.lineWidth = lineWidth
      this.context.lineColor = lineColor
    },
    setColor () {
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
canvas {
  border: 1px solid black;
}
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
