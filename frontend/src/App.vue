<template>
  <p v-if="!isSupported">This browser won't work, please try another</p>
  <div v-else>
    <button v-if="microphoneAccess !== 'granted'" @click="microphonePrompt">
      Enable microphone use
    </button>
    <button v-else-if="microphoneAccess == 'granted' && !shouldPlay" @click="shouldPlay = true">
      Play
    </button>
    <div v-else-if="shouldPlay" class="video-container">
      <video autoplay ref="videoPlayer" @ended="listenToPassword" :src="videoSource" :poster="'test'"></video>
    </div>

    <Present v-if="passwordCorrect" />

  </div>
</template>

<script setup>
import { ref, watch } from 'vue';
import { useSpeechRecognition } from '@vueuse/core';
import { usePermission } from '@vueuse/core'
import Present from './Present.vue';

const { result, start, stop, isSupported } = useSpeechRecognition();

const startVideoSource = '/password-cut.mp4';
const endVideoSource = '/end.mp4';

const videoSource = ref(startVideoSource);
const passwordCorrect = ref(false);
const shouldPlay = ref(false);

const microphoneAccess = usePermission('microphone')

const listenToPassword = async () => {
  start()
}

const checkPassword = (result) => {
  const fuzzyMatch = ["kapoot", "kappa", "patric", "carpet", "draconus", "draconis"];

  if (fuzzyMatch.some(sub => result.value.includes(sub))) {
    passwordCorrect.value = true;
    videoSource.value = endVideoSource;
    stop();
  }
}

const microphonePrompt = () => {
  start();
  stop();
}

watch(result, (newResult) => {
  checkPassword(newResult)
});

</script>

<style scoped>
.video-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  z-index: 10;
}

video {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.container {
  display: flex;
  flex-direction: column;
  width: 100vw;
  height: 100vh;
}
</style>