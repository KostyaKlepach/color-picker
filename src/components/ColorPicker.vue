<template>
  <div class="text-xs-center">
    <v-dialog v-model="dialog" width="300">
      <template v-slot:activator="{ on }">
        <v-btn dark v-on="on">Color Picker</v-btn>
      </template>

      <v-card>
        <v-card-title primary-title>Vue Color pickers</v-card-title>
        <div div class="color-picker">
          <div class="color-picker-elevation d-inline-flex justify-center">
            <label for="color-input" id="color-label" style="background-color: red"></label>
            <input type="checkbox" id="color-input" checked />

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
            <!-- <input :value="color" type="color" @change="setPickerColor($event)" /> -->
            <input
              class="slider"
              type="range"
              min="0"
              max="1"
              step="0.1"
              @change="setOpacity($event)"
              id="opacity"
              value="1"
            />
            <!-- :style="'background:'+color" -->
            <span>{{colorOpacity * 100 + '%'}}</span>
          </div>
          <!-- <v-flex xs12>
            <v-slider :thumb-size="20"  thumb-label="always" />
          </v-flex>-->

          <div class="color-picker-swatches">
            <div
              class="color"
              v-for="(color, i) in colors"
              :key="i"
              :style="'background:'+color"
              @click="setSwatchesColor(color)"
            ></div>
          </div>
        </div>
        <v-card-actions class="d-inline-flex">
          <v-spacer></v-spacer>
          <v-btn dark @click="resetColor">Cancel</v-btn>
          <v-btn dark @click="dialog=false">OK</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>
<script>
export default {
  name: 'color-picker',
  props: ['colors', 'color'],
  data () {
    return {
      dialog: false,
      colorOpacity: 1,
      currentColor: '',
      localValue: this.value,
      drage: false,
      rgbaColor: 'rgba(255,0,0,1)',
      pickerWidth: '',
      pickerHeight: '',
      contentPicker: '',
      contentRainbow: '',
      x: 0,
      y: 0,
      colorLabel: ''
    };
  },
  methods: {
    setSwatchesColor (color) {
      this.$emit('setColor', this.hexToRGB(color))
    },
    setPickerColor (event) {
      this.$emit('setColor', this.hexToRGB(event.target.value))
    },
    setOpacity (event) {
      this.colorOpacity = event.target.value;
    },
    hexToRGB (color) {
      let r = parseInt(color.slice(1, 3), 16),
        g = parseInt(color.slice(3, 5), 16),
        b = parseInt(color.slice(5, 7), 16);
      return `rgba( ${r}, ${g} , ${b}, ${this.colorOpacity})`;
    },
    resetColor () {
      this.$emit('setColor', '#F0F0F0')
      this.dialog = false
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
  },
}
</script>
<style scoped>
.color-picker {
  display: grid;
  align-items: center;
}

.color-picker-elevation {
  margin-top: 12px;
}

.color-picker-swatches {
  width: 84%;
  display: grid;
  margin: 16px auto;
  grid-template-columns: repeat(5, 52px);
}
.color {
  border: 2px solid #fff;
  height: 28px;
  border-radius: 8px;
}

input[type='color'] {
  -webkit-appearance: none;
  border: none;
  width: 52px;
  height: 28px;
  border-radius: 8px;
}

.slider {
  -webkit-appearance: none;
  width: 210px;
  margin: auto 10px;
  border-radius: 8px;
  outline: none;
  background: linear-gradient(
    87deg,
    rgba(202, 243, 233, 1) 10%,
    rgba(202, 243, 233, 1) 25%,
    rgba(141, 255, 227, 1) 40%,
    rgba(11, 230, 209, 0.8) 60%,
    rgba(23, 179, 164, 0.9) 89%
  );
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
