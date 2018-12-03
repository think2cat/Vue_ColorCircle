<template>
  <div class="main">
    <div class="farbtastic">
      <div class="wheel"></div>
      <div class="h-marker marker"></div>
    </div>
    <ul>
      <li>
        <img src="./images/symbols-panel_saturation_bright.png" alt="">
        <input type="range" min="0" max="100" v-model="sat" />
        <img src="./images/symbols-panel_saturation_original-color.png" alt="">
      </li>
      <li>
        <img src="./images/symbols-panel_brightness_dark.png" alt="">
        <input type="range" min="0" max="100" v-model="lum" />
        <img src="./images/symbols-panel_brightness_bright.png" alt="">
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  data() {
    return {
      e: undefined,
      whell: undefined,
      marker: undefined,
      radius: 60,
      square: 60,
      width: 130,
      color: undefined,
      hsl: [],
      rgb: undefined,
      dragging: false,
      sat: 100,
      lum: 50
    }
  },
  mounted() {
    this.e = this.$el.querySelector('.farbtastic')
    this.wheel = this.$el.querySelector('.wheel')
    this.marker = this.$el.querySelector('.marker')

    this.e.style.width = this.width + 'px'
    this.e.style.height = this.width + 'px'
    this.wheel.style.width = this.width + 'px'
    this.wheel.style.height = this.width + 'px'

    // $._farbtastic = (container, callback) => {
      // Store farbtastic object
      // let fb = this

      // let e = $('.farbtastic', container)
      // fb.wheel = $('.wheel', container).get(0)
      // Dimensions
      // fb.radius = 60
      // fb.square = 60
      // fb.width = 120

      /**
       * Link to the given element(s) or callback.
       */
      // fb.linkTo = callback => {
      //   // Unbind previous nodes
      //   if (typeof fb.callback == 'object') {
      //     $(fb.callback).unbind('keyup', fb.updateValue)
      //   }

      //   // Reset color
      //   fb.color = null

      //   // Bind callback or elements
      //   if (typeof callback == 'function') {
      //     fb.callback = callback
      //   } else if (typeof callback == 'object' || typeof callback == 'string') {
      //     fb.callback = $(callback)
      //     fb.callback.bind('keyup', fb.updateValue)
      //     if (fb.callback.get(0).value) {
      //       fb.setColor(fb.callback.get(0).value)
      //     }
      //   }
      //   return this
      // }
      // fb.updateValue = event => {
      //   if (this.value && this.value != fb.color) {
      //     fb.setColor(this.value)
      //   }
      // }





      // Install mousedown handler (the others are set on the document on-demand)
      // $('*', e).mousedown(fb.mousedown)

      // Init color
    //   fb.setColor('#fff')

    //   // Set linked elements/callback
    //   if (callback) {
    //     fb.linkTo(callback)
    //   }
    // }

    // $.fn.farbtastic = function(callback) {
    //   $.farbtastic(this, callback)
    //   return this
    // }
    // $.farbtastic = function(container, callback) {
    //   var container = $(container).get(0);
    //   return container.farbtastic || (container.farbtastic = new $._farbtastic(container, callback));
    // }
    // $(this.$el).farbtastic()
    this.e.addEventListener('mousedown', this.mousedown)
    this.setColor('#126930')
    // $('*', e).mousedown(fb.mousedown)
  },
  watch: {
    sat() {
      this.setHSL([this.hsl[0], this.sat/100, this.lum/100])
    },
    lum() {
      this.setHSL([this.hsl[0], this.sat/100, this.lum/100])
    }
  },
  methods: {
    /**
     * Change color with HSL triplet [0..1, 0..1, 0..1]
     */
    setHSL(hsl){
      // console.log('setHSL', hsl)
      if (this.hsl[0] + this.hsl[1] + this.hsl[1] === 0 && hsl[0] !== 0) {
        this.sat = 100
        this.lum = 50
      }
      this.hsl = hsl
      this.color = this.pack(this.HSLToRGB(this.hsl))
      this.updateDisplay()
    },
    /**
     * Mousemove handler
     */
    mousemove(event) {
        // Get coordinates relative to color picker center
        let pos = this.widgetCoords(event)

        // Set new HSL parameters
        // if (this.circleDrag) {
          let hue = Math.atan2(pos.x, -pos.y) / 6.28
          if (hue < 0) hue += 1
          this.setHSL([hue, this.hsl[1], this.hsl[2]])
        // } else {
        //   let sat = Math.max(0, Math.min(1, -(pos.x / this.square) + 0.5))
        //   let lum = Math.max(0, Math.min(1, -(pos.y / this.square) + 0.5))
        //   //TODO HSL转16进制
          // this.setHSL([hue, this.sat, this.lum])
        // }
        return false
      },
      /**
       * Mouseup handler
       */
      mouseup() {
        // Uncapture mouse
        this.e.removeEventListener('mousemove', this.mousemove)
        this.e.removeEventListener('mouseup', this.mouseup)
        this.dragging = false
      },
      /**
       * Mousedown handler
       */
      mousedown(event) {
        // Capture mouse
        if (!this.dragging) {
          this.e.addEventListener('mousemove', this.mousemove)
          this.e.addEventListener('mouseup', this.mouseup)
          this.dragging = true
        }

        // Check which area is being dragged
        // let pos = this.widgetCoords(event)
        // Process
        this.mousemove(event)
        return false
      },
      /**
       * Change color with HTML syntax #123456
       */
      setColor(color) {
        // console.log('setColor', color)
        this.color = color
        let unpack = this.unpack(color)
        // if (this.color != color && unpack) {
          // this.rgb = unpack
          this.hsl = this.RGBToHSL(unpack)
          this.sat = Math.round(this.hsl[1] * 100)
          this.lum = Math.round(this.hsl[2] * 100)
          this.updateDisplay()
        // }
      },
      /**
       * Retrieve the coordinates of the given event relative to the center
       * of the widget.
       */
      widgetCoords(event) {
        // console.log('widgetCoords', event)
        let x, y
        // let el = event.target || event.srcElement
        let reference = this.wheel

        // if (false && typeof event.offsetX != 'undefined') {
        //   // Use offset coordinates and find common offsetParent
        //   let pos = { x: event.offsetX, y: event.offsetY }

        //   // Send the coordinates upwards through the offsetParent chain.
        //   this.e = el
        //   while (this.e) {
        //     console.log('this.e1', this.e)
        //     this.e.mouseX = pos.x
        //     this.e.mouseY = pos.y
        //     pos.x += this.e.offsetLeft
        //     pos.y += this.e.offsetTop
        //     this.e = this.e.offsetParent
        //   }

        //   // Look for the coordinates starting from the wheel widget.
        //   this.e = reference
        //   let offset = { x: 0, y: 0 }
        //   while (this.e) {
        //     console.log('this.e2', this.e)
        //     if (typeof this.e.mouseX != 'undefined') {
        //       x = this.e.mouseX - offset.x
        //       y = this.e.mouseY - offset.y
        //       break
        //     }
        //     offset.x += this.e.offsetLeft
        //     offset.y += this.e.offsetTop
        //     this.e = this.e.offsetParent
        //   }

        //   // Reset stored coordinates
        //   this.e = el
        //   while (this.e) {
        //     console.log('this.e3', this.e)
        //     this.e.mouseX = undefined
        //     this.e.mouseY = undefined
        //     this.e = this.e.offsetParent
        //   }
        // } else {
          // Use absolute coordinates
          let pos = this.absolutePosition(reference)
          x =
            (event.pageX || 0 * (event.clientX + document.body.scrollLeft)) - pos.x
          y =
            (event.pageY || 0 * (event.clientY + document.body.scrollTop)) - pos.y
        // }
        // Subtract distance to middle
        // console.log('widgetCoords.return', { x: x - this.width / 2, y: y - this.width / 2 })
        return { x: x - this.width / 2, y: y - this.width / 2 }
      },
      /**
       * Update the markers and styles
       */
      updateDisplay() {
        // Markers
        let angle = this.hsl[0] * 6.28
        this.marker.style.left = Math.round(Math.sin(angle) * this.radius + this.width / 2) + 'px'
        this.marker.style.top = Math.round(-Math.cos(angle) * this.radius + this.width / 2) + 'px'

        this.$emit('change', this.color)
      },
      /**
       * Get absolute position of element
       */
      absolutePosition(el) {
        // console.log('absolutePosition', el)
        let r = { x: el.offsetLeft, y: el.offsetTop }
        // Resolve relative to offsetParent
        if (el.offsetParent) {
          let tmp = this.absolutePosition(el.offsetParent)
          r.x += tmp.x
          r.y += tmp.y
        }
        // console.log('absolutePosition.return', r)
        return r
      },
      /* letious color utility functions */
      pack(rgb) {
        // console.log('pack', rgb)
        let r = Math.round(rgb[0] * 255)
        let g = Math.round(rgb[1] * 255)
        let b = Math.round(rgb[2] * 255)
        return (
          '#' +
          (r < 16 ? '0' : '') +
          r.toString(16) +
          (g < 16 ? '0' : '') +
          g.toString(16) +
          (b < 16 ? '0' : '') +
          b.toString(16)
        )
      },
      // 16进制转HSL
      unpack(color) {
        // console.log('unpack', color)
        if (color.length == 7) {
          return [
            parseInt('0x' + color.substring(1, 3)) / 255,
            parseInt('0x' + color.substring(3, 5)) / 255,
            parseInt('0x' + color.substring(5, 7)) / 255
          ]
        } else if (color.length == 4) {
          return [
            parseInt('0x' + color.substring(1, 2)) / 15,
            parseInt('0x' + color.substring(2, 3)) / 15,
            parseInt('0x' + color.substring(3, 4)) / 15
          ]
        }
      },
      HSLToRGB(hsl) {
        // console.log('HSLToRGB', hsl)
        let m1, m2	// , r, g, b
        let h = hsl[0],
          s = hsl[1],
          l = hsl[2]
        m2 = l <= 0.5 ? l * (s + 1) : l + s - l * s
        m1 = l * 2 - m2
        // console.log('HSLToRGB.return', [
        //   this.hueToRGB(m1, m2, h + 0.33333),
        //   this.hueToRGB(m1, m2, h),
        //   this.hueToRGB(m1, m2, h - 0.33333)
        // ])
        return [
          this.hueToRGB(m1, m2, h + 0.33333),
          this.hueToRGB(m1, m2, h),
          this.hueToRGB(m1, m2, h - 0.33333)
        ]
      },
      hueToRGB(m1, m2, h) {
        // console.log('hueToRGB', m1, m2, h)
        h = h < 0 ? h + 1 : h > 1 ? h - 1 : h
        if (h * 6 < 1) return m1 + (m2 - m1) * h * 6
        if (h * 2 < 1) return m2
        if (h * 3 < 2) return m1 + (m2 - m1) * (0.66666 - h) * 6
        return m1
      },
      RGBToHSL(rgb) {
        // console.log('RGBToHSL', rgb)
        let min, max, delta, h, s, l
        let r = rgb[0],
          g = rgb[1],
          b = rgb[2]
        min = Math.min(r, Math.min(g, b))
        max = Math.max(r, Math.max(g, b))
        delta = max - min
        l = (min + max) / 2
        s = 0
        if (l > 0 && l < 1) {
          s = delta / (l < 0.5 ? 2 * l : 2 - 2 * l)
        }
        h = 0
        if (delta > 0) {
          if (max == r && max != g) h += (g - b) / delta
          if (max == g && max != b) h += 2 + (b - r) / delta
          if (max == b && max != r) h += 4 + (r - g) / delta
          h /= 6
        }
        // console.log('RGBToHSL.return', [h, s, l])
        return [h, s, l]
      }
  }
}
</script>

<style lang="less" scoped>
.main {
	width: 150px;
}
.farbtastic,
.farbtastic .wheel {
  margin: 0 auto;
}

.farbtastic {
  width: 100%;
  position: relative;
  .wheel {
    background-image: url('images/wheel.png');
    background-repeat: no-repeat;
    background-size: 100% 100%;
  }
  .marker {
    width: 17px;
    height: 17px;
    margin: -8px 0 0 -8px;
    overflow: hidden;
    background-image: url('images/marker.png');
    background-repeat: no-repeat;
    position: absolute;
    cursor: crosshair;
    z-index: 1000;
  }
}
// .farbtastic* {
  // position: absolute;
  // cursor: crosshair;
// }

ul {
  width: 100%;
  & > li {
    margin-top: 8px;
    overflow: hidden;
    & > img {
      float: left;
    }
    & > input[type='range'] {
      float: left;
    }
  }
}
</style>