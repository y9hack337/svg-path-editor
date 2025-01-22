<template>
  <div id="app" class="spe">
    <header class="spe-header">
      <h1 class="spe-heading">SVG Path Editor</h1>
      <div class="spe-controls">
        <label class="spe-toggle">
          <input type="checkbox" class="spe-toggle__checkbox" v-model="showGrid">
          <div class="spe-toggle__btn"></div>
          <div class="spe-toggle__label">Show grid</div>
        </label>
         <div class="spe-zoom-controls">
          <button @click="zoom(0.1)">+</button>
          <button @click="zoom(-0.1)">-</button>
        </div>
      </div>
    </header>
    <section class="spe-main">
      <aside class="spe-aside">
        <div class="spe-aside__box">
          <label class="spe-label" for="input">Input path</label>
          <textarea id="input" class="spe-textarea" :class="{'spe-textarea--error': inputPathError}" v-model="inputPath"></textarea>
          <div class="spe-error" v-if="inputPathError">Path is not valid</div>
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="scale">Scale</label>
          <input id="scale" class="spe-input" type="number" min="0.1" step="0.01" v-model="options.scale">
        </div>
        <div class="spe-aside__box">
          <div class="spe-label">Translate</div>
          <div class="spe-aside__group">
            <div class="spe-aside__combo">
              <label class="spe-aside__combo-label" for="translateX">X:</label>
              <input id="translateX" class="spe-input" type="number" step="0.1" v-model="options.translateX">
            </div>
            <div class="spe-aside__combo">
              <label class="spe-aside__combo-label" for="translateY">Y:</label>
              <input id="translateY" class="spe-input" type="number" step="0.1" v-model="options.translateY">
            </div>
          </div>
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="rotate">Rotate</label>
          <input id="rotate" class="spe-input" type="number" step="0.1" v-model="options.rotate">
        </div>
        <div class="spe-aside__box">
          <label class="spe-label" for="output">Output path</label>
          <textarea id="output" class="spe-textarea" v-model="outputPath" onclick="this.focus();this.select()" readonly></textarea>
        </div>
      </aside>
      <div class="spe-content" :style="{ transform: `scale(${zoomLevel})` }">
        <svg version="1.1"
          baseProfile="full"
          width="100%" height="100%"
          xmlns="http://www.w3.org/2000/svg"
          class="spe-content__svg">
          <!-- Grid -->
          <defs>
            <pattern id="smallGrid" width="10" height="10" patternUnits="userSpaceOnUse">
              <path d="M 10 0 L 0 0 0 10" fill="none" stroke="#393939" opacity="0.4" stroke-width="0.5"/>
            </pattern>
            <pattern id="grid" width="100" height="100" patternUnits="userSpaceOnUse">
              <rect width="100" height="100" fill="url(#smallGrid)"/>
              <path d="M 100 0 L 0 0 0 100" fill="none" stroke="#393939" opacity="0.4" stroke-width="1"/>
            </pattern>
          </defs>
          <rect class="spe-content__svg-grid" :class="{'spe-content__svg-grid--visible': showGrid}" width="100%" height="100%" fill="url(#grid)" />
          <!-- Path -->
          <path :d="outputPath" fill="#393939" ref="path"></path>
        </svg>
      </div>
    </section>
  </div>
</template>

<script>
import svgpath from 'svgpath';

export default {
  name: 'app',
  data () {
    return {
      inputPath: "M164.441 44.563l53.795 53.794-136.169 136.17-53.765-53.794 136.14-136.171z m94.616-12.975l-23.991-23.99c-9.271-9.271-24.326-9.271-33.629 0l-22.981 22.98 53.795 53.795 26.806-26.805c7.191-7.192 7.191-18.789 0-25.98z m-258.907 224.757c-0.979 4.406 2.999 8.354 7.405 7.282l59.946-14.534-53.765-53.795-13.586 61.047z",
      inputPathError: false,
      outputPath: null,
      options: {
        scale: 1,
        translateX: 0,
        translateY: 0,
        rotate: 0
      },
      showGrid: true,
      centerX: 0,
      centerY: 0,
      zoomLevel: 1
    }
  },
  methods: {
    updatePath() {
      const newPath = svgpath(this.inputPath);
      if(!newPath.err && this.inputPath){
        this.outputPath = newPath
                          .rotate(parseFloat(this.options.rotate) || 0, this.centerX, this.centerY)
                          .scale(parseFloat(this.options.scale) || 1)
                          .translate(parseFloat(this.options.translateX) || 0, parseFloat(this.options.translateY) || 0)
                          .rel()
                          .round(3)
                          .toString();
        this.inputPathError = false;
      } else {
        this.inputPathError = true;
      }
    },
    resetOptions() {
      this.options = {
        scale: 1,
        translateX: 0,
        translateY: 0,
        rotate: 0
      }
    },
    setCenterCoordinates() {
      const SVGRect = this.$refs.path.getBBox();
      this.centerX = SVGRect.width / 2 + SVGRect.x;
      this.centerY = SVGRect.height / 2 + SVGRect.y;
    },
    zoom(factor) {
      this.zoomLevel += factor;
    }
  },
  watch: {
    inputPath() {
      this.resetOptions();
      this.updatePath();
      setTimeout(() => {
        this.setCenterCoordinates();
      }, 100);
    },
    options: {
      handler(options) {
        if(options.scale < 0) options.scale = 1;
        this.updatePath();
      },
      deep: true
    }
  },
  mounted() {
    this.updatePath();
    setTimeout(() => {
      this.setCenterCoordinates();
    }, 100);
  }
}
</script>
<style scoped lang="scss">
/* ... (previous styles) ... */
.spe-zoom-controls {
  display: inline-flex;
  button {
    margin-left: 5px;
  }
}

</style>
