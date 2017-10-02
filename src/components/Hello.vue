<template>
  <div class='hello'>
    <canvas id="canvas" width="300" height="300"></canvas>
  </div>
</template>

<script>
export default {
  name: 'hello',
  data () {
    return {
      msg: 'Welcome to Your Vue.js App'
    }
  },
  mounted () {
    (function draw () {
      var canvas = document.getElementById('canvas')
      if (canvas.getContext) {
        var ctx = canvas.getContext('2d')
        ctx.beginPath()
        ctx.arc(100, 100, 99, 2 * Math.PI, false)

        ctx.moveTo(194, 100)
        ctx.arc(100, 100, 94, 0, 2* Math.PI, false)

        ctx.translate(100, 100)
        ctx.rotate(1)
        ctx.moveTo(0,0)
        ctx.lineTo(0, -85)
        ctx.stroke()
      }
    })()
  }
}
</script>

<!-- Add 'scoped' attribute to limit CSS to this component only -->
<style scoped>
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
