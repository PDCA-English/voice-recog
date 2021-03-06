<template>
  <div id="app">
    <v-app id="inspire">
      <v-container fluid>
        <v-layout row wrap justify-center class="mt-4">
          <v-flex xs12 sm10 text-xs-center>
            <v-text-field
              label="The text"
              v-model="text"
              textarea
            ></v-text-field>
          </v-flex>
          <v-flex xs12 sm8 md4 text-xs-center>
            <speech-to-text :text.sync="text" @speechend="speechEnd"></speech-to-text>
          </v-flex>
          <v-flex xs12 text-xs-center class="mt-4">
            {{sentences}}
          </v-flex>
        </v-layout>
      </v-container>
    </v-app>
  </div>
</template>

<style>
</style>


<script src="https://unpkg.com/babel-polyfill/dist/polyfill.min.js"></script>
<script src="https://unpkg.com/vuetify@1.0.17/dist/vuetify.min.js"></script>
<script type="module">
  import Vue from 'https://cdn.jsdelivr.net/npm/vue@2.6.12/dist/vue.esm.browser.js'


let SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition
let recognition = SpeechRecognition? new SpeechRecognition() : false

Vue.component('speech-to-text', {
  template: `
    <v-card>
    <v-card-text>
      <v-layout row wrap justify-space-around>
        <v-flex xs8 sm9 text-xs-center>
          <p v-if="error" class="grey--text">{{error}}</p>
          <p v-else class="mb-0">
            <span v-if="sentences.length > 0" v-for="sentence in sentences">{{sentence}}. </span>
            <span>{{runtimeTranscription}}</span>
          </p>
        </v-flex>
        <v-flex xs2 sm1 text-xs-center>
          <v-btn
            dark
            @click.stop="toggle ? endSpeechRecognition() : startSpeechRecognition()"
            icon
            :color="!toggle ? 'grey' : (speaking ? 'red' : 'red darken-3')"
            :class="{'animated infinite pulse': toggle}"
          >
            <v-icon>{{toggle ? 'mic_off' : 'mic'}}</v-icon>
          </v-btn>
        </v-flex>
      </v-layout>
    </v-card-text>
  </v-card>
  `,
  props: {
    lang: {
      type: String,
      default: 'es-ES'
    },
    text: {
      type: [String, null],
      required: true
    }
  },
  data () {
    return {
      error: false,
      speaking: false,
      toggle: false,
      runtimeTranscription: '',
      sentences: []
    }
  },
  methods: {
    checkCompatibility () {
      if (!recognition) {
        this.error = 'Speech Recognition is not available on this browser. Please use Chrome or Firefox'
      }
    },
    endSpeechRecognition () {
      recognition.stop()
      this.toggle = false
      this.$emit('speechend', {
        sentences: this.sentences,
        text: this.sentences.join('. ')
      })
    },
    startSpeechRecognition () {
      if (!recognition) {
        this.error = 'Speech Recognition is not available on this browser. Please use Chrome or Firefox'
        return false
      }
      this.toggle = true
      recognition.lang = this.lang
      recognition.interimResults = true

      recognition.addEventListener('speechstart',() => {
        this.speaking = true
      })

      recognition.addEventListener('speechend', () => {
        this.speaking = false
      })

      recognition.addEventListener('result', event => {
        const text = Array.from(event.results).map(result => result[0]).map(result => result.transcript).join('')
        this.runtimeTranscription = text
      })

      recognition.addEventListener('end', () => {
        if (this.runtimeTranscription !== '') {
          this.sentences.push(this.capitalizeFirstLetter(this.runtimeTranscription))
          this.$emit('update:text', `${this.text}${this.sentences.slice(-1)[0]}. `)
        }
        this.runtimeTranscription = ''
        recognition.stop()
        if (this.toggle) {
          // keep it going.
          recognition.start()
        }
      })
      recognition.start()
    },
    capitalizeFirstLetter (string) {
      return string.charAt(0).toUpperCase() + string.slice(1)
    }
  },
  mounted () {
    this.checkCompatibility()
  }
})

new Vue({
  el: '#app',
  data () {
    return {
      text: '',
      sentences: null
    }
  },
  methods: {
    speechEnd ({sentences, text}) {
      console.log('text', text)
      console.log('sentences', sentences)
      this.sentences = sentences
    }
  }
})
</script>




