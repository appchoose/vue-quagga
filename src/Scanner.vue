<template>
  <div id="interactive" class="viewport scanner">
    <video></video>
    <canvas class="drawingBuffer"></canvas>
  </div>
</template>

<script>
import Quagga from "quagga";

export default {
  name: "scanner",
  props: {
    onDetected: {
      type: Function
    },
    onStop: {
      type: Function
    },
    onStart: {
      type: Function
    },
    onProcessed: {
      type: Function
    },
    readerType: {
      type: String,
      default: "code_128_reader"
    },
    multiple: {
      type: Boolean,
      default: false
    },
    singleChannel: {
      type: Boolean,
      default: false
    },
    frequency: {
      type: Number,
      default: 20
    },
    readerSize: {
      width: {
        type: Number,
        default: 640
      },
      height: {
        type: Number,
        default: 480
      }
    }
  },
  data: function() {
    return {
      quaggaState: {
        inputStream: {
          type: "LiveStream",
          constraints: {
            width: { min: this.readerSize.width },
            height: { min: this.readerSize.height },
            facingMode: "environment",
            aspectRatio: { min: 1, max: 2 }
          }
        },
        locator: {
          patchSize: "medium",
          halfSample: true
        },
        singleChannel: this.singleChannel,
        multiple: this.multiple,
        numOfWorkers: 2,
        frequency: this.frequency,
        decoder: {
          readers: [
            {
              format: this.readerType,
              config: {}
            }
          ]
        },
        locate: true
      }
    };
  },
  mounted: function() {
    Quagga.init(this.quaggaState, function(err) {
      if (err) {
        return console.log(err);
      }
    });
    Quagga.onDetected(this.onDetected ? this.onDetected : this._onDetected);
    Quagga.onProcessed(this.onProcessed ? this.onProcessed : this._onProcessed);
  },
  methods: {
    start: function() {
      Quagga.stop();
    },
    stop: function() {
      Quagga.stop();
    },
    _onProcessed: function(result) {
      let drawingCtx = Quagga.canvas.ctx.overlay,
        drawingCanvas = Quagga.canvas.dom.overlay;

      if (result) {
        if (result.boxes) {
          drawingCtx.clearRect(
            0,
            0,
            parseInt(drawingCanvas.getAttribute("width")),
            parseInt(drawingCanvas.getAttribute("height"))
          );
          result.boxes
            .filter(function(box) {
              return box !== result.box;
            })
            .forEach(function(box) {
              Quagga.ImageDebug.drawPath(box, { x: 0, y: 1 }, drawingCtx, {
                color: "green",
                lineWidth: 2
              });
            });
        }
        if (result.box) {
          Quagga.ImageDebug.drawPath(result.box, { x: 0, y: 1 }, drawingCtx, {
            color: "#00F",
            lineWidth: 2
          });
        }

        if (result.codeResult && result.codeResult.code) {
          Quagga.ImageDebug.drawPath(
            result.line,
            { x: "x", y: "y" },
            drawingCtx,
            { color: "red", lineWidth: 3 }
          );
        }
      }
    },
    _onDetected: function(result) {
      console.log("detected: ", result);
    }
  }
};
</script>

<style scoped>
.viewport {
  position: relative;
}

.viewport canvas,
.viewport video {
  position: absolute;
  left: 0;
  top: 0;
}
</style>
