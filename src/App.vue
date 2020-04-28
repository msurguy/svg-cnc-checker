<template>
  <div id="app">
    <div class="page">
      <!-- Sidebar -->
      <div class="sidebar">
        <div class="controls-wrapper">
          <div class="controls">
            <p class="mt-3 lead text-center text-white"> Source </p>
            <div class="sidebar-control">
              <div class="control-header">
                <div></div>
                <div class="control-label">
                  Upload File
                </div>
                <div></div>
              </div>
              <div class="custom-file">
                <input @change="handleFileUpload" type="file" ref="file" accept="image/svg+xml"
                       class="custom-file-input"
                       id="sourcefile">
                <label class="custom-file-label" for="sourcefile">Upload SVG file</label>
              </div>
              <button class="btn btn-primary btn-sm" @click="submitFile">Submit</button>
            </div>

            <div class="progress" v-if="source.loading">
              <div class="progress-bar" role="progressbar"
                   aria-valuenow="100" aria-valuemin="0" aria-valuemax="100" style="width: 100%">Loading SVG
              </div>
            </div>

            <transition name="slide">
              <div v-if="source.svg">
               
              </div>
            </transition>

          </div>
        </div>

        <div class="button">
          <div class="reveal"></div>
          <button class="btn btn-primary btn-block" @click.prevent="download">
            Download SVG
          </button>
        </div>
      </div>

      <!-- Page Content -->
      <div class="paper">
        <div id="sketch" class="sketch">
          <div id="svg-wrapper" ref="svgWrapper">
          </div>
        </div>
      </div>
      <div class="footer-wrapper">
        <div class="footer">
          <h3>SVG Checker</h3>
          <p>Project by <a target="_blank" href="http://twitter.com/msurguy">@msurguy</a> (<a target="_blank"
                                                                                              href="http://github.com/msurguy/cnc-text-tool">Source</a>)
          </p>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import bsCustomFileInput from 'bs-custom-file-input'

  import Toggle from "@/components/Toggle";
  import SelectField from "@/components/SelectField";
  import Slider from "@/components/Slider";
  // import ButtonGroup from "@/components/ButtonGroup";
  import TextAreaInput from "@/components/TextAreaInput";
  import DoubleTextInput from "@/components/DoubleTextInput";

  import { downloadSVG } from "@/util/download";

  export default {
    name: 'Rooot',
    components: {
      Toggle,
      SelectField,
      Slider,
      TextAreaInput,
      DoubleTextInput
    },
    props: {
      msg: String
    },
    data () {
      return {
        file: '',
        source: {
          filename: '',
          svg: '',
          string: '',
          loading: false,
          width: 0,
          height: 0,
          viewbox: '0 0 0 0'
        }
      }
    },
    mounted () {
      bsCustomFileInput.init();
    },
    methods: {
        /*
        Submits the file to the server
      */
      submitFile(){
        /*
                Initialize the form data
            */
            let formData = new FormData();

            /*
                Add the form data we need to submit
            */
            formData.append('file', this.file);
            /*
              Additional POST Data
            */
            // formData.append('first_name', 'Dan');
            // formData.append('last_name', 'Pastori');
        /*
          Make the request to the POST /single-file URL
        */
            this.axios.post( 'https://svg-optimizer-api.msurguy.now.sh/api/optimize',
                formData,
                {
                headers: {
                  'Access-Control-Allow-Origin': '*',
                    'Content-Type': 'multipart/form-data; charset=utf-8;'
                }
              }
            ).then(function(response){
          console.log(response);
        })
        .catch(function(error){
          console.log(error);
        });
      },

      /*
        Handles a change on the file upload
      */
      handleFileUpload(){
        this.file = this.$refs.file.files[0];
      },
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
      download () {
        // downloadSVG(this.$refs.svgWrapper.childNodes[1], flattenedGroup, `line-text-${Date.now()}`)
      }
    }
  }
</script>

<style>

  #app {
    height: 100%;
  }

  .canvas {
    position: absolute;
    left: 0;
    top: 0;
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
    position: relative;
    /*overflow: auto;*/
  }

  #svg-wrapper {
    position: relative;
    display: inline-block;
    border: 1px dashed #000;
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
</style>

