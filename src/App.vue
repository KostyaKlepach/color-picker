<template>
  <v-app :style="'background:'+color">
    <!-- <label for="color-input" id="color-label" style="background-color: red"></label>
    <input type="checkbox" id="color-input" checked />-->

    <div id="color-picker">
      <canvas
        id="color-block"
        @mousedown="startColorSelection"
        @mouseup="finishColorSelection"
        @mousemove="chooseColor"
        height="150"
        width="150"
      ></canvas>
      <canvas id="color-strip" @click="chooseGradient" height="150" width="30"></canvas>
    </div>
    <ColorPicker class="button" :color="color" :colors="colors" @setColor="setColor" />
  </v-app>
</template>

<script>

import ColorPicker from '@/components/ColorPicker';

export default {
  name: 'app',
  components: {
    ColorPicker
  },
  data () {
    return {
      colors: [
        '#ef9a9a', '#e57373', '#ef5350', '#f44336', '#f48fb1',
        '#f06292', '#ec407a', '#e91e63', '#ce93d8', '#ba68c8',
        '#ab47bc', '#9c27b0', '#b39ddb', '#9575cd', '#7e57c2',
        '#673AB7', '#9FA8DA', '#7986CB', '#5C6BC0', '#3F51B5',
        '#90CAF9', '#64B5F6', '#42A5F5', '#2196F3', '#81D4FA',
        '#4FC3F7', '#29B6F6', '#03A9F4', '#80DEEA', '#4DD0E1',
        '#26C6DA', '#00BCD4', '#80CBC4', '#4DB6AC', '#26A69A',
        '#009688', '#A5D6A7', '#81C784', '#66BB6A', '#4CAF50',
        '#C5E1A5', '#AED581', '#9CCC65', '#8BC34A', '#E6EE9C',
        '#DCE775', '#D4E157', '#CDDC39', '#FFC107', '#ffb407',
      ],

      color: '#F0F0F0',
      colorOpacity: 1,
      drage: false,
      rgbaColor: 'rgba(255,0,0,1)',
      pickerWidth: '',
      pickerHeight: '',
      contentPicker: '',
      contentRainbow: '',
      x: 0,
      y: 0,
      colorLabel: ''
    }
  },
  mounted () {
    this.setCanvas()
  },
  methods: {
    setColor (color) {
      this.color = color;
    },
    setCanvas () {
      let colorBlock = document.getElementById('color-block');
      this.contentPicker = colorBlock.getContext('2d'); // contentPicker
      this.pickerWidth = colorBlock.width;
      this.pickerHeight = colorBlock.height;

      let colorStrip = document.getElementById('color-strip');
      this.contentRainbow = colorStrip.getContext('2d'); // contentRainbow
      let rainbowWidth = colorStrip.width;
      let rainbowHeight = colorStrip.height;

      this.colorLabel = document.getElementById('color-label');

      this.contentPicker.rect(0, 0, this.pickerWidth, this.pickerHeight)
      this.fillGradient()

      this.contentRainbow.rect(0, 0, rainbowWidth, rainbowHeight);
      let gradient = this.contentRainbow.createLinearGradient(0, 0, 0, this.pickerHeight)
      gradient.addColorStop(0, 'rgba(255, 0, 0, 1)')
      gradient.addColorStop(0.17, 'rgba(255, 255, 0, 1)')
      gradient.addColorStop(0.34, 'rgba(0, 255, 0, 1)')
      gradient.addColorStop(0.51, 'rgba(0, 255, 255, 1)')
      gradient.addColorStop(0.68, 'rgba(0, 0, 255, 1)')
      gradient.addColorStop(0.85, 'rgba(255, 0, 255, 1)')
      gradient.addColorStop(1, 'rgba(255, 0, 0, 1)')
      this.contentRainbow.fillStyle = gradient
      this.contentRainbow.fill()
    },
    chooseGradient (e) {
      this.x = e.offsetX;
      this.y = e.offsetY;
      let imageData = this.contentRainbow.getImageData(this.x, this.y, 1, 1).data
      this.rgbaColor = 'rgba(' + imageData[0] + ',' + imageData[1] + ',' + imageData[2] + ',1)'
      this.fillGradient()
    },
    fillGradient () {
      this.contentPicker.fillStyle = this.rgbaColor
      this.contentPicker.fillRect(0, 0, this.pickerWidth, this.pickerHeight)

      let grdWhite = this.contentRainbow.createLinearGradient(0, 0, this.pickerWidth, 0);
      grdWhite.addColorStop(0, 'rgba(255,255,255,1)')
      grdWhite.addColorStop(1, 'rgba(255,255,255,0)')
      this.contentPicker.fillStyle = grdWhite;
      this.contentPicker.fillRect(0, 0, this.pickerWidth, this.pickerHeight)

      let grdBlack = this.contentRainbow.createLinearGradient(0, 0, 0, this.pickerHeight)
      grdBlack.addColorStop(0, 'rgba(0,0,0,0)')
      grdBlack.addColorStop(1, 'rgba(0,0,0,1)')
      this.contentPicker.fillStyle = grdBlack
      this.contentPicker.fillRect(0, 0, this.pickerWidth, this.pickerHeight)
    },
    startColorSelection (e) {
      this.drag = true
      this.changeColor(e)
    },
    chooseColor (e) {
      if (this.drag) {
        this.changeColor(e)
      }
    },
    finishColorSelection (e) {
      this.drag = false
    },
    changeColor (e) {
      this.x = e.offsetX
      this.y = e.offsetY
      let imageData = this.contentPicker.getImageData(this.x, this.y, 1, 1).data
      this.rgbaColor = 'rgba(' + imageData[0] + ',' + imageData[1] + ',' + imageData[2] + ',1)'
      this.colorLabel.style.backgroundColor = this.rgbaColor
      this.color = this.rgbaColor
    }
  }
}
</script>

<style scoped>
body {
  margin: 0;
  padding: 0;
  background: #f3f3f3;
}

#app {
  text-align: center;
  color: #2c3e50;
}

h2 {
  background-color: #dbdbdb;
  margin: 0;
  margin-bottom: 15px;
  padding: 10px;
  font-family: 'Open Sans';
}

#color-input {
  display: none;
}

#color-label {
  margin-left: 15px;
  position: absolute;
  height: 30px;
  width: 50px;
}

#color-input:checked ~ #color-picker {
  opacity: 1;
}

#color-picker {
  display: flex;
  position: absolute;
  left: 70px;
  height: 150px;
  width: 185px;
  opacity: 1;
  padding: 5px;
}

#color-strip {
  margin-left: 10px;
}

canvas:hover {
  cursor: crosshair;
}
</style>

