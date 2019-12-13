<template>
  <v-container>
    <v-layout
      text-center
      wrap
    >
      <v-flex mb-4>
        <h2>
          Camera
          <div id="webcam-container"></div>
        </h2>
      </v-flex>

      <v-flex
        mb-5
        xs12
      >
        <h2 class="headline font-weight-bold mb-3">
          <div id="label-container"></div>
        </h2>
      </v-flex>

    </v-layout>
  </v-container>
</template>

<script>
import * as tmImage from '@teachablemachine/image'

let model, webcam, labelContainer, maxPredictions
const URL = './my_model/' //'https://raw.githubusercontent.com/gaomar/teachable-machine-model-demo/master/'
export default {
  name: 'HelloWorld',
  data () {
    return {

    }
  },
  created () {
    this.init()
  },
  methods: {
    init: async function() {
      const modelURL = URL + "model.json"
      const metadataURL = URL + "metadata.json"

      model = await tmImage.load(modelURL, metadataURL)
      maxPredictions = model.getTotalClasses()

      // Convenience function to setup a webcam
      const flip = true // whether to flip the webcam
      webcam = new tmImage.Webcam(640, 480, flip) // width, height, flip
      await webcam.setup() // request access to the webcam
      await webcam.play()
      requestAnimationFrame(this.loop)

      // append elements to the DOM
      document.getElementById("webcam-container").appendChild(webcam.canvas)
      labelContainer = document.getElementById("label-container")
      for (let i = 0; i < maxPredictions; i++) { // and class labels
        labelContainer.appendChild(document.createElement("div"))
      }
    },
    loop: async function() {
      webcam.update(); // update the webcam frame
      await this.predict();
      requestAnimationFrame(this.loop)
    },
    predict: async function() {
      const prediction = await model.predict(webcam.canvas)
      for (let i = 0; i < maxPredictions; i++) {
        const classPrediction = prediction[i].className + ": " + prediction[i].probability.toFixed(2)
        labelContainer.childNodes[i].innerHTML = classPrediction
      }
    }
  }
}
</script>
