<template>
  <v-app>
    <div class="app">
      <v-container>
        <v-row>
          <v-col>
            <v-slider
              max="0"
              min="-50"
              step="0.1"
              vertical
              label="Threshold"
              v-model="threshold"
              v-on:change="onThresholdChange"
            ></v-slider>
          </v-col>
          <v-col align-self="center">
            <div class="threshold-text">
              Value: {{ threshold }} <br />
              Raise the threshold value until ambient noise is no longer tracked
            </div>
          </v-col>
        </v-row>

        <v-row>
          <v-btn elevation="2" v-on:click="startMicrophoneButtonPress"
            >Start Microphone</v-btn
          >
          <v-btn elevation="2" v-on:click="stopMicrophoneButtonPress"
            >Stop Microphone</v-btn
          >
        </v-row>

        <v-row class="sentence">
          <div>
            Sentence in morse code: {{ morseString }}<br />
            Sentence in hangul: {{ hangulSentence }}
          </div>
        </v-row>

        <v-row>
          <v-btn elevation="2" v-on:click="clearSentenceButtonPress"
            >Clear Sentence</v-btn
          >
        </v-row>

        <v-row>
          <v-col>
            <div
              class="square"
              v-bind:style="{ backgroundColor: activeColor }"
            ></div>
          </v-col>
          <v-col>
            <div>
              White square: A word delimiter will be added<br />
              Grey square: A character delimiter will be added<br />
              Red square: Audio is being "heard"<br />
              Blue square: Adding a dot if the audio stops<br />
              Green square: Adding a dash if the audio stops
            </div>
          </v-col>
        </v-row>
      </v-container>
    </div>
  </v-app>
</template>

<script>
import MicToMorseCode from "mic-to-morse-code";

export default {
  name: "App",

  data: () => ({
    threshold: -50,
    activeColor: "white",
    morseString: "",
    hangulSentence: "",
  }),

  methods: {
    startMicrophoneButtonPress: function () {
      micToMorseCode.startMicrophone();
    },
    stopMicrophoneButtonPress: function () {
      micToMorseCode.stopMicrophone();
    },
    clearSentenceButtonPress: function () {
      micToMorseCode.clearSentence();
    },
    onThresholdChange: function (value) {
      micToMorseCode.setThreshold(value);
    },
  },

  mounted: function () {
    createReferenceToVueInstance(this);
  },
};
const eventToColorMap = {
  "not-listening": "white",
  "listening:no-sound": "yellow",
  "listening:sound": "red",
  "dot:length": "blue",
  "dash:length": "green",
  "character:delimiter:length": "grey",
  "word:delimiter:length": "white",
};
let vue = null;

function createReferenceToVueInstance(vueInstance) {
  vue = vueInstance;
}

const micToMorseCode = new MicToMorseCode(-50, 50, 0.2);

micToMorseCode.createListener("on:change", function (morseSentence) {
  vue.morseString = morseSentence;
  vue.hangulSentence = getHangulSentence(vue.morseString);
});

micToMorseCode.createListener("on:audio:state:change", function (audioState) {
  vue.activeColor = eventToColorMap[audioState];
});

function getHangulSentence(string) {
  return string
    .split("/")
    .map((morseWord) => {
      return morseWord
        .split(" ")
        .map((morseCharacter) => {
          return convertMorseCodeToHangul(morseCharacter);
        })
        .join(""); // Should do a Hangul.assemble here, too
    })
    .join(" ");
}

function convertMorseCodeToHangul(characterInMorseCode) {
  /**
   * convert!
   */
  return characterInMorseCode;
}
</script>

<style>
.row {
  padding-top: 15px;
}

.v-btn {
  margin-right: 10px;
}
.app {
  padding: 10px;
  width: 800px;
}
.v-input {
  width: 300px;
}
.v-slider {
  height: 250px;
}

.sentence {
  min-height: 30px;
}

.square {
  margin: 10px;
  height: 100px;
  width: 100px;
}
</style>
