<template>
  <div id="app">
    <div class="page">
      <!-- Sidebar -->
      <div class="sidebar">
        <div class="controls-wrapper">
          <div class="controls">
            <div class="sidebar-control">
              <div class="control-header">
                <div></div>
                <div class="control-label">
                  Choose SVG File
                </div>
                <div></div>
              </div>
              <div class="custom-file">
                <input
                  id="sourcefile"
                  type="file"
                  accept="image/svg+xml"
                  class="custom-file-input"
                  @change="handleFileUpload">
                <label
                  class="custom-file-label"
                  for="sourcefile">Upload SVG file</label>
              </div>
              <button
                class="btn btn-primary btn-sm"
                @click="submitFile">
                Upload
                <transition name="fade">
                  <span
                    v-if="source.loading"
                    class="spinner-border spinner-border-sm"
                    role="status"
                    aria-hidden="true"></span>
                </transition>
              </button>
            </div>

            <transition name="slide">
              <div v-if="optimizedSVG">
                <text-input
                  v-model="strokeWidth"
                  label="Stroke Width"></text-input>

                <div class="sidebar-control">
                  <div class="zoom-controls">
                    <span class="small">zoom: <kbd>alt</kbd> + <kbd>mousewheel</kbd></span>
                    <div class="buttons">
                      <button
                        class="btn btn-sm"
                        @click="zoomIn">
                        <svg
                          id="i-zoom-in"
                          xmlns="http://www.w3.org/2000/svg"
                          viewBox="0 0 32 32"
                          width="16"
                          height="16"
                          fill="none"
                          stroke="currentcolor"
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          stroke-width="2">
                          <circle
                            cx="14"
                            cy="14"
                            r="12" />
                          <path d="M23 23 L30 30" />
                          <path d="M14 10 L14 18 M10 14 L18 14" />
                        </svg>
                      </button>
                      <button
                        class="btn btn-sm"
                        @click="zoomOut">
                        <svg
                          id="i-zoom-out"
                          xmlns="http://www.w3.org/2000/svg"
                          viewBox="0 0 32 32"
                          width="16"
                          height="16"
                          fill="none"
                          stroke="currentcolor"
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          stroke-width="2">
                          <circle
                            cx="14"
                            cy="14"
                            r="12" />
                          <path d="M23 23 L30 30" />
                          <path d="M10 14 L18 14" />
                        </svg>
                      </button>
                      <button
                        class="btn btn-sm pr-0"
                        @click="resetZoom">
                        <svg
                          id="i-zoom-reset"
                          xmlns="http://www.w3.org/2000/svg"
                          viewBox="0 0 32 32"
                          width="16"
                          height="16"
                          fill="none"
                          stroke="currentcolor"
                          stroke-linecap="round"
                          stroke-linejoin="round"
                          stroke-width="2">
                          <circle
                            cx="14"
                            cy="14"
                            r="12" />
                          <path d="M23 23 L30 30" />
                          <path d="M9 12 L9 9 12 9 M16 9 L19 9 19 12 M9 16 L9 19 12 19 M19 16 L19 19 16 19" />
                        </svg>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </transition>
          </div>
        </div>

        <!-- <div class="button">
          <div class="reveal"></div>
          <button
            class="btn btn-primary btn-block"
            @click.prevent="download">
            Download SVG
          </button>
        </div> -->
      </div>

      <!-- Page Content -->
      <div class="paper">
        <div
          id="sketch"
          class="sketch">
          <div
            id="svg-wrapper"
            ref="svgWrapper"
            v-html="optimizedSVG">
          </div>
        </div>
      </div>
      <div class="footer-wrapper">
        <div class="footer">
          <h3>SVG Checker</h3>
          <p>
            Project by <a
              target="_blank"
              href="http://twitter.com/msurguy">@msurguy</a> (<a
                target="_blank"
                href="http://github.com/msurguy/cnc-text-tool">Source</a>)
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import bsCustomFileInput from 'bs-custom-file-input'
import panzoom from 'panzoom'
// import Toggle from '@/components/Toggle'
import TextInput from '@/components/TextInput'
// import { downloadSVG } from '@/util/download'

export default {
  name: 'Rooot',
  components: {
    TextInput
  },
  data () {
    return {
      file: '',
      strokeWidth: '0.2mm',
      optimizedSVG: null,
      source: {
        element: null,
        loading: false,
        width: 0,
        height: 0,
        viewbox: '0 0 0 0'
      },
      panzoom: null
    }
  },
  watch: {
    strokeWidth (val) {
      this.source.element.setAttribute('stroke-width', val)
    }
  },
  mounted () {
    bsCustomFileInput.init()
  },
  methods: {
    zoomIn () {
      this.panzoom.smoothZoom(0, 0, 1.5)
    },
    zoomOut () {
      this.panzoom.smoothZoom(0, 0, 0.5)
    },
    resetZoom () {
      this.panzoom.moveTo(0, 0)
      this.panzoom.zoomAbs(0, 0, 1)
    },
    /*
        Submits the file to the server
      */
    submitFile () {
      this.source.loading = true
      /*
                Initialize the form data
            */
      const formData = new FormData()

      /*
                Add the form data we need to submit
            */

      formData.append('file', this.file)
      /*
              Additional POST Data
            */
      const optionsJSON = {
        addDefaultFillStroke: true,
        precision: 2,
        removeDimensions: false
      }
      formData.append('flags', JSON.stringify(optionsJSON))
      this.axios.post('https://svg-optimizer-api.msurguy.now.sh/api/optimize',
        formData,
        {
          headers: {
            'Access-Control-Allow-Origin': '*',
            'Content-Type': 'multipart/form-data; charset=utf-8;'
          }
        }
      ).then(({ data }) => {
        this.optimizedSVG = data
        // Update dimensions of the SVG here
        // Get width and height from ViewBox if not present otherwise!
        this.source.loading = false
        this.$nextTick(() => {
          this.source.element = this.$refs.svgWrapper.firstChild
          const renderedSVG = this.source.element
          // add width and height if they're missing
          if (renderedSVG.getAttribute('viewBox') && !renderedSVG.getAttribute('width') && !renderedSVG.getAttribute('height')) {
            const dimensions = renderedSVG.getAttribute('viewBox').split(/[ ,]+/g)
            renderedSVG.setAttribute('width', dimensions[2])
            renderedSVG.setAttribute('height', dimensions[3])
          }
          this.strokeWidth = '0.2mm'

          this.panzoom = panzoom(this.source.element, {
            smoothScroll: false,
            beforeWheel: function (e) {
              // allow wheel-zoom only if altKey is down. Otherwise - ignore
              var shouldIgnore = !e.altKey
              return shouldIgnore
            }
          })
        })
      })
        .catch((error) => {
          console.log(error)
          this.source.loading = false
        })
    },

    /*
        Handles a change on the file upload
      */
    handleFileUpload (e) {
      const files = e.target.files || e.dataTransfer.files
      if (!files.length) { return }

      if (this.panzoom) { this.panzoom.dispose() }

      const reader = new FileReader()
      reader.onload = (e) => {
        this.file = new Blob([e.target.result.replace(/([+]?\d+\.\d{3,}([eE][+]?\d+)?)/g, (x) => (+x).toFixed(2))], { type: 'image/svg+xml;charset=utf-8' })
      }
      reader.readAsText(files[0])
    }
    // onSourceFileChange (e) {
    //   this.source.loading = true;
    //   // reset position / scale / other params of the text
    //   let files = e.target.files || e.dataTransfer.files;
    //   if (!files.length)
    //     return;
    //   this.readSourceImage(files[0]);
    // },
    // readSourceImage (file) {
    //   const reader = new FileReader();
    //   reader.onload = async (e) => {
    //     // Set file name
    //     this.source.filename = file.name.substr(0, file.name.lastIndexOf('.'));

    //     // try finding "<svg" in the document:
    //     let fileContents = e.target.result;
    //     this.processInputSVG(fileContents)
    //   };
    //   reader.readAsText(file);
    // },
    // async processInputSVG (fileContents) {
    //   console.log(fileContents)
    // },
    // download () {
    //   // downloadSVG(this.$refs.svgWrapper.childNodes[1], flattenedGroup, `line-text-${Date.now()}`)
    // }
  }
}
</script>

<style>

  #app {
    height: 100%;
  }

  #svg-wrapper rect {
    pointer-events: all;
  }

  #svg-wrapper svg:first-child rect:hover {
    fill: rgba(0, 0, 0, 0.08)
  }

  .page {
    position: relative;
    height: 100%;
    display: flex;
  }

  .zoom-controls {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
  }

  .controls {
    width: 100%;
    margin-bottom: 50px;
  }

  .controls-wrapper {
    max-height: 100vh;
    overflow: scroll;
  }

  .button {
    position: absolute;
    bottom: 0;
    width: 100%;
    text-align: center;
  }

  .reveal {
    display: block;
    height: 15px;
    background: linear-gradient(to bottom, rgba(0, 0, 0, 0) 0%, rgb(47, 47, 47) 100%);
  }

  .paper {
    padding: 10px;
    background-color: #dedede;
    position: relative;
    max-height: 100vh;
    width: calc(100% - 300px);
    overflow: scroll;
    z-index: 1;
  }

  .sketch {
    z-index: 1;
    position: relative;
    /*overflow: auto;*/
  }

  #svg-wrapper {
    position: relative;
    display: inline-block;
    border: 1px dashed #000;
    overflow: hidden;
  }

  .sidebar {
    z-index: 10;
    width: 300px;
    position: relative;
  }

  .footer-wrapper {
    z-index: 1000;
    position: absolute;
    bottom: 0;
    right: 0;
    color: #2D2D2D;
  }

  .footer {
    padding: 15px 15px 0 15px;
    text-align: right;
  }

  @media (max-width: 767px) {
    .page {
      flex-direction: column;
    }

    .controls-wrapper {
      max-height: none;
    }

    .sidebar {
      width: 100%;
    }

    .paper {
      width: 100%;
      max-height: none;
    }

    .footer-wrapper {
      position: relative;
      background-color: #CCC;
    }
  }

  .slide-enter-active,
  .slide-leave-active {
    transition: all 300ms ease-in-out;
  }

  .slide-enter-to,
  .slide-leave {
    max-height: 200px;
    opacity: 1;
    overflow: hidden;
  }

  .slide-enter,
  .slide-leave-to {
    max-height: 0;
    opacity: 0;
    overflow: hidden;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition: all 300ms;
  }

  .fade-enter-to,
  .fade-leave {
    opacity: 1;
  }

  .fade-enter,
  .fade-leave-to {
    opacity: 0;
  }
</style>
