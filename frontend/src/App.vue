<template>
  <p v-if="!isSupported">This browser won't work, please try another</p>
  <div v-else>
    <button v-if="microphoneStatus == 'granted' && !shouldPlay" @click="shouldPlay = true">
      Play
    </button>
    <div v-else-if="shouldPlay" class="video-container">
      <video autoplay ref="videoPlayer" @ended="listenToPassword" :src="videoSource" :poster="'test'"></video>
    </div>

    <Present v-if="passwordCorrect" />

  </div>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue';
import { useSpeechRecognition } from '@vueuse/core';
import Present from './Present.vue';

const { result, start, stop, isSupported } = useSpeechRecognition();

const startVideoSource = '/password-cut.mp4';
const endVideoSource = '/end.mp4';

const videoSource = ref(startVideoSource);
const passwordCorrect = ref(false);
const shouldPlay = ref(false);

const microphoneStatus = ref('idle');

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

onMounted(async () => {
  await checkAndRequestMic();
})

const getPermissionStatus = async () => {
  if (!navigator.permissions) {
    // Permissions API not supported
    return null;
  }
  
  try {
    const result = await navigator.permissions.query({ name: 'microphone' });
    return result.state; 
    
  } catch (error) {
    console.error("Error querying microphone permissions:", error);
    return null;
  }
}

const checkAndRequestMic = async () => {
  const status = await getPermissionStatus();

  if (status === 'granted') {
    // Permission is already granted, proceed without prompting
    microphoneStatus.value = 'granted';
    console.log("Permission already granted.");
    return;
  }

  // If status is 'prompt' or 'denied' (or if Permissions API is not supported), 
  // we attempt to get the stream, which will trigger the prompt if needed.
  microphoneStatus.value = 'pending';
  const granted = await requestMicrophoneAccess();
  microphoneStatus.value = granted ? 'granted' : 'denied';
}

// A simple function to request access
const requestMicrophoneAccess = async () => {
  try {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    stream.getTracks().forEach(track => track.stop());
    return true; // Return true for success

  } catch (error) {
    console.error("Microphone access denied or error:", error.name, error.message);

    if (error.name === 'NotAllowedError' || error.name === 'PermissionDeniedError') {
      alert("Microphone permission was denied. Please allow it in your browser settings.");
    } else if (error.name === 'NotFoundError') {
      alert("No microphone found on your device.");
    }

    return false; // Return false for failure
  }
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