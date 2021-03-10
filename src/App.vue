<template>
  <v-app>
    <div class="app">
      <v-container>
        <v-row>
          <v-col>
            <v-slider
              max="0"
              min="-40"
              step="0.1"
              vertical
              label="Threshold"
              v-model="threshold"
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
          <v-btn elevation="2" v-on:click="startMicrophone"
            >Start Microphone</v-btn
          >
          <v-btn elevation="2" v-on:click="stopMicrophone"
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
          <v-btn elevation="2" v-on:click="clearSentence">Clear Sentence</v-btn>
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
import * as Tone from "tone";

export default {
  name: "App",

  data: () => ({
    threshold: -40,
    activeColor: "white",
    morseString: "",
    hangulSentence: "",
  }),

  methods: {
    startMicrophone: function () {
      if (Tone.Transport.state !== "started") {
        Tone.start();
      }

      const meter = new Tone.Meter();
      microphoneSource = new Tone.UserMedia().connect(meter);
      microphoneSource
        .open()
        .then(() => {
          playTone();
          listenInterval = setInterval(onMicTime(this, meter), 100);
        })
        .catch((e) => {
          console.log(e);
        });
    },
    stopMicrophone: function () {
      clearInterval(listenInterval);
      microphoneSource.close();
      this.activeColor = "white";
    },
    clearSentence: function () {
      morseSentence = "";
    },
  },
};

const debounceTime = 0.25;
const dotTime = 0.5;
const dashTime = 1.5;
const letterTime = 1.5;
const spaceTime = 2.5;
const letterDelimiter = " ";
const spaceDelimiter = "/";

let listenInterval = null;
let soundStartTime = 0;
let soundStopTime = 0;
let morseSentence = "";
let microphoneSource = null;
let lastState = 0;

function getHangulSentence() {
  return morseSentence
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

function getMorseString() {
  return morseSentence;
}

function playTone() {
  const synth = new Tone.Synth().toDestination();
  synth.triggerAttackRelease("C4", "8n");
}

function onMicTime(self, meter) {
  return function () {
    const currentTime = meter.now();
    if (meter.getValue() > self.threshold) {
      logStartTime(self, currentTime);
      return;
    }
    logEndTime(self, currentTime);
    self.hangulSentence = getHangulSentence();
    self.morseString = getMorseString();
  };
}

function logStartTime(self, currentTime) {
  if (currentTime - soundStopTime < debounceTime) {
    return;
  }
  if (wasOn()) {
    const onTime = currentTime - soundStartTime;
    if (onTime > dashTime) {
      self.activeColor = "green";
      return;
    }
    if (onTime > dotTime) {
      self.activeColor = "blue";
      return;
    }
    return;
  }
  self.activeColor = "red";
  lastState = 1;
  soundStartTime = currentTime;
}

function wasOn() {
  return lastState === 1;
}

function logEndTime(self, currentTime) {
  if (currentTime - soundStartTime < debounceTime) {
    return;
  }
  const offTime = currentTime - soundStopTime;
  if (wasOff() && offTime > spaceTime) {
    self.activeColor = "white";
    if (morseSentence.length === 0) {
      return;
    }
    if (morseSentence[morseSentence.length - 1] === spaceDelimiter) {
      return;
    }
    addCharacter(spaceDelimiter);
    return;
  }
  if (wasOff() && offTime > letterTime) {
    self.activeColor = "grey";
    if (morseSentence.length === 0) {
      return;
    }
    if (morseSentence[morseSentence.length - 1] === " ") {
      return;
    }
    addCharacter(letterDelimiter);
    return;
  }

  if (wasOn()) {
    soundStopTime = currentTime;
    lastState = 0;
    const onTime = currentTime - soundStartTime;
    if (onTime > dashTime) {
      addCharacter("-");
      return;
    }
    if (onTime > dotTime) {
      addCharacter(".");
      return;
    }
  }
}

function wasOff() {
  return lastState === 0;
}

function addCharacter(character) {
  morseSentence = morseSentence + character;
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
