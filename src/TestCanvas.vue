<template>
  <div>
    <div>
      Polyline is <span>{{ polylinePoints }}</span>.
    </div>
    <svg width="500" height="500" @mousemove="onCanvasMouseMove" @mouseup="onCanvasMouseUp"
         @mousedown="onCanvasMouseDown"
    >
      <circle v-for="p in points" :cx="p.x" :cy="p.y"
              @mouseover="mouseOverPoint(p)" @mouseout="mouseOutPoint(p)"
              @mousedown="startMovePoint($event, p)"
              @click.prevent="onPointClick($event, p)"
              :r="p === highlightedPoint ? 12: 10" stroke="green" stroke-width="3" fill="yellow"></circle>
      <polygon :points="polylinePoints" fill="none" stroke="blue"></polygon>
      <text v-for="l in letters" :x="l.x" :y="l.y" font-family="Verdana" font-size="20"
            text-anchor="middle" dominant-baseline="middle" style="-webkit-user-select: none;">{{ l.text }}</text>
    </svg>
  </div>
</template>

<script>
function grc (event, element) {
  element = element || event.currentTarget
  const rect = element.getBoundingClientRect()
  return {
    x: event.clientX - rect.left,
    y: event.clientY - rect.top
  }
}

function cutInBetween (p1, p2, cutFactor) {
  return {
    x: p1.x + ((p2.x - p1.x) * cutFactor),
    y: p1.y + ((p2.y - p1.y) * cutFactor)
  }
}

function pixelFromRay (p1, p2, distPx) {
  const cutFactor = distPx / Math.sqrt(Math.pow(p2.x - p1.x, 2) + Math.pow(p2.y - p1.y, 2))
  return cutInBetween(p1, p2, cutFactor)
}

export default {
  name: 'test-canvas',
  props: ['points'],
  data: function () {
    return {
      highlightedPoint: null,
      pointAtMove: null,
      mouseToTargetDelta: { x: 0, y: 0 },
      mousePos: { x: 0, y: 0 }
    }
  },
  computed: {
    resting: function () {
      return !this.pointAtMove
    },
    polylinePoints: function () {
      return this.points.map(p => `${p.x},${p.y}`).join(' ')
    },
    letters: function () {
      return this.points.map((p, i) => {
        const median = cutInBetween(this.points[this.pidx(i - 1)], this.points[this.pidx(i + 1)], 0.5)
        const pos = pixelFromRay(this.points[i], median, 20)
        return {
          text: String.fromCharCode(65 + i),
          x: pos.x,
          y: pos.y
        }
      })
    }
  },
  methods: {
    pidx: function (idx) {
      idx = idx % this.points.length
      idx = (idx < 0) ? this.points.length + idx : idx
      return idx
    },
    mouseOverPoint: function (point) {
      if (this.resting) {
        this.highlightedPoint = point
      }
    },
    mouseOutPoint: function (point) {
      if (this.highlightedPoint === point) {
        this.highlightedPoint = null
      }
    },
    startMovePoint: function (event, point) {
      if (event.button !== 0) {
        return
      }
      this.pointAtMove = point
      this.mouseToTargetDelta.x = point.x - this.mousePos.x
      this.mouseToTargetDelta.y = point.y - this.mousePos.y
    },
    onCanvasMouseMove: function (event) {
      const where = grc(event)
      this.mousePos.x = where.x
      this.mousePos.y = where.y

      if (this.pointAtMove) {
        this.pointAtMove.x = where.x + this.mouseToTargetDelta.x
        this.pointAtMove.y = where.y + this.mouseToTargetDelta.y
      }
    },
    onCanvasMouseUp: function () {
      this.pointAtMove = null
    },
    onCanvasMouseDown: function (event) {
      if (event.button !== 0) {
        return
      }
      // if clicked on canvas
      if (event.target === event.currentTarget) {
        const where = grc(event)
        this.points.push(where)
      }
    },
    onPointClick: function (event, point) {
      if (event.button === 1) {
        this.points.splice(this.points.indexOf(point), 1)
      }
    }
  }
}
</script>

<style scoped>
  svg { background-color: azure }
  circle { color: red; }
</style>
