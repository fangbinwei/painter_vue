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
        <section class="pic-generator"
                 @click="getImage">生成图像</section>
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
    <div id="img-box">
      <div class="img-item" 
           v-for="(item,index) in imgUrl"
           :key="index">
        <img :src="item" alt="">
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
      imgUrl: [],
      shapes: ['line', 'curve', 'rect', 'circle'],
      colors: ['black', 'red', 'pink', 'blue', 'tomato'],
      canvasMouseMove: false,
      context: {},
      trackContext: {},
      config: {
        lineShape: 'curve',
        lineWidth: 2,
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
      preArray: [],
      curArray: [],
      nextArray: [],
      shapeStart: {
        x: 0,
        y: 0
      },
      shapeEnd: {
        x: 0,
        y: 0
      },
      circleDataTemp: {
        centerX: 0,
        centerY: 0,
        r: 0
      }
    }
  },
  methods: {
    // 初始化获取canvas元素
    init () {
      this.canvas = document.getElementById('canvas')
      this.canvasTrack = document.getElementById('canvas-track')
      this.context = this.canvas.getContext('2d')
      // this.context.translate(0.5, 0.5)
      this.trackContext = this.canvasTrack.getContext('2d')
      // this.trackContext.translate(0.5, 0.5)
    },
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
    // 画图鼠标按下事件
    canvasDown (e) {
      this.setCanvasStyle()
      this.drawStrategies(this.config.lineShape).canvasDown.call(this, e)
      // save previous data
      this.saveData(this.context, this.preArray)
    },
    // 画图鼠标松开事件
    canvasUp (e) {
      this.drawStrategies(this.config.lineShape).canvasUp.call(this, e)
      // save current data
      this.curArray.length = 0
      this.saveData(this.context, this.curArray)
    },
    // 画图鼠标移动事件
    canvasMove (e) {
      this.drawStrategies(this.config.lineShape).canvasMove.call(this, e)
    },
    // 画图操作
    canvasControl (action, e) {
      switch(action) {
        case 'prev':
          this.prev(this.context, this.preArray, this.curArray, this.nextArray)
          break
        case 'next':
          this.next(this.context, this.preArray, this.curArray, this.nextArray)
          break
        case 'clear':
          this.clear()
          break
      }
    },
    prev (context, preArr, curArray, nextArr) {
      if (preArr.length) {
        // save current data into nextArr
        nextArr.push(curArray.pop())

        // pop previous data to render the canvas
        let popData = preArr.pop()
        context.putImageData(popData, 0, 0)

        // save previous data into curArr
        curArray.push(popData)
      }
    },
    next (context, preArr, curArray, nextArr) {
      if (nextArr.length) {
        // save current data into preArr
        preArr.push(curArray.pop())

        // pop next data to render the canvas
        let popData = nextArr.pop()
        context.putImageData(popData, 0, 0)

        // save next data into currArr
        curArray.push(popData)
      }
    },
    // 获取鼠标在canvas中的位置
    getMousePos (e, canvas) {
      let rect = canvas.getBoundingClientRect()
      // canvasX = e.clientX - e.target.offsetLeft
      // canvasY = e.clientY - e.target.offsetTop
      return {
        x: e.clientX - rect.left,
        y: e.clientY - rect.top
      }

    },
    // 清除轨迹canvas内容
    clearTrack () {
      this.trackContext.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
    },
    // 清除canvas内容
    clear () {
      if (!this.canvasMouseMove) {
        this.context.clearRect(0, 0, this.context.canvas.width, this.context.canvas.height)
        this.clearArrayTemp ()
      }
    },
    clearArrayTemp () {
      this.preArray.length = 0
      this.curArray.length = 0
      this.nextArray.length = 0
    },
    // 计算鼠标位置作为起始点
    shapeStartCal (e) {
      this.shapeStart.x = this.getMousePos(e, this.canvas).x
      this.shapeStart.y = this.getMousePos(e, this.canvas).y
    },
    // 计算鼠标位置作为结束点
    shapeEndCal (e) {
      this.shapeEnd.x = this.getMousePos(e, this.canvas).x
      this.shapeEnd.y = this.getMousePos(e, this.canvas).y

    },
    drawLine (context, clearFlag, startX, startY, endX, endY) {
      if (clearFlag) {
        context.clearRect(0, 0, this.canvas.width, this.canvas.height)
        console.log(context)
      }
      context.beginPath()
      context.moveTo(startX, startY)
      context.lineTo(endX, endY)
      context.stroke()
    },
    drawRect (context, clearFlag, startX, startY, endX, endY) {
      let w, h, width, height, offsetX, offsetY
      w = endX - startX
      h = endY - startY
      // 矩阵起始点偏移
      offsetX = (w < 0) ? w : 0
      offsetY = (h < 0) ? h : 0
      // 矩阵宽 高
      width = Math.abs(endX - startX)
      height = Math.abs(endY - startY)
      // 是否清除轨迹
      if (clearFlag) {
        context.clearRect(0, 0, this.canvas.width, this.canvas.height)
      }
      context.beginPath()
      context.rect(startX + offsetX, startY + offsetY, width, height)
      context.stroke()
    },
    drawCircle (context, clearFlag, x, y, r, sAngle=0, eAngle=2*Math.PI, counterclockwise=true) {
      if (clearFlag) {
        context.clearRect(0, 0, this.canvas.width, this.canvas.height)
      }
      context.beginPath()
      context.arc(x, y, r, sAngle, eAngle, counterclockwise)
      context.stroke()
    },
    saveData (context, statusArray, width=this.canvas.width, height=this.canvas.height) {
      let data = context.getImageData(0, 0, width, height)
      statusArray.push(data)
    },
    getImage () {
      let canvas = this.canvas
      let src = canvas.toDataURL('imge/png')
      this.imgUrl.push(src)
    },
    /**
     * @param {string} type 'curve' or 'rect' and etc.
     */
    drawStrategies (type) {
      switch(type) {
        case 'line':
          return {
            canvasDown (e) {
              console.log('line down')
              if (!this.canvasMouseMove) {
                this.canvasMouseMove = true
                // 获取直线起点坐标
                this.shapeStartCal(e)
              }
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                // 获取直线终点坐标
                this.shapeEndCal(e)
                // 在轨迹canvas中显示轨迹,并实时清除上一次的轨迹
                this.drawLine(this.trackContext, true, this.shapeStart.x, this.shapeStart.y, this.shapeEnd.x, this.shapeEnd.y)
                console.log('line move')
              }

            },
            canvasUp (e) {
              this.canvasMouseMove = false
              // 清理轨迹canvas中的轨迹
              this.clearTrack()
              console.log('line up')
              this.drawLine(this.context, false, this.shapeStart.x, this.shapeStart.y, this.shapeEnd.x, this.shapeEnd.y)

            }
          }
        case 'curve':
          return {
            canvasDown (e) {
              this.canvasMouseMove = true
              this.context.beginPath()
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                console.log('canvasMove')
                // 获取当前鼠标位置的坐标,用于画曲线
                this.shapeStartCal(e)
                console.log(e.clientX,this.shapeStart.x, this.shapeStart.y)
                this.context.lineTo(this.shapeStart.x, this.shapeStart.y)
                this.context.stroke()
                console.log(this.context.strokeStyle)
              }
            },
            canvasUp (e) {
              this.canvasMouseMove = false
            }
          }
        case 'rect':
          return {
            canvasDown (e) {
              if (!this.canvasMouseMove) {
                this.canvasMouseMove = true
                // 获取矩形左上角坐标
                this.shapeStartCal(e)
                console.log('rect down')
              }
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                // 获取矩形右下角坐标
                this.shapeEndCal(e)
                // 在轨迹canvas中显示轨迹,并实时清除上一次轨迹
                this.drawRect(this.trackContext, true, this.shapeStart.x, this.shapeStart.y, this.shapeEnd.x, this.shapeEnd.y)
              console.log('rect move')
              }
            },
            canvasUp (e) {
              this.canvasMouseMove = false
              // 清理轨迹canvas的内容
              this.clearTrack()
              this.drawRect(this.context, false, this.shapeStart.x, this.shapeStart.y, this.shapeEnd.x, this.shapeEnd.y)
              console.log('rect up')
            }
          }
        case 'circle':
          return {
            canvasDown (e) {
              if (!this.canvasMouseMove) {
                this.canvasMouseMove = true
                this.shapeStartCal(e)
                console.log('circle', this.shapeStart.x, this.shapeStart.y)
              }
            },
            canvasMove (e) {
              if (this.canvasMouseMove) {
                this.shapeEndCal(e)
                console.log('circle', this.shapeEnd.x, this.shapeEnd.y)
                let centerX, centerY, r
                this.circleDataTemp.centerX = centerX = (this.shapeEnd.x + this.shapeStart.x)/2
                this.circleDataTemp.centerY = centerY= (this.shapeEnd.y + this.shapeStart.y)/2
                this.circleDataTemp.r = r = Math.sqrt((this.shapeEnd.x - this.shapeStart.x)*
                              (this.shapeEnd.x - this.shapeStart.x)+
                              (this.shapeEnd.y - this.shapeStart.y)*
                              (this.shapeEnd.y - this.shapeStart.y))/2
                console.log(centerX, centerY, r)
                this.drawCircle(this.trackContext, true, centerX, centerY, r)
              }

            },
            canvasUp (e) {
              this.canvasMouseMove = false
              // 清理轨迹canvas的内容
              this.clearTrack()
              this.drawCircle(this.context, false, this.circleDataTemp.centerX, this.circleDataTemp.centerY, this.circleDataTemp.r)
              console.log('circle up')

            }
          }
      }
    }
  },
  mounted () {
    // 获取dom中canvas元素
    this.init()
    // 初始化canvas相关参数
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
.pic-generator {
  cursor: pointer;
}

/* canvas */
canvas {
  border: 1px solid black;
  cursor: crosshair;
  position: absolute;
}
.canvas-wrap {
  height: 400px;
}
#canvas {
  z-index: 2;
}
#canvas-track {
  z-index: 3;
}
</style>
