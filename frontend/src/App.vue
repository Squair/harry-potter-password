<template>
  <div>
    <button v-if="microphoneStatus == 'granted' && !shouldPlay" @click="shouldPlay = true">
      Play
    </button>
    <div v-else-if="shouldPlay && !secondVidFinished" class="video-container">
      <video v-show="!passwordCorrect" autoplay ref="videoPlayer" @ended="toggleListening" :src="startVideoSource" :poster="'test'"></video>
      <video preload="auto" v-show="passwordCorrect" :autoplay="passwordCorrect" ref="videoPlayer2" @ended="secondVidFinished = true" :src="endVideoSource" :poster="'test'"></video>
    </div>

    <Present v-if="secondVidFinished" />

  </div>
</template>

<script setup>
import { onMounted, ref, watch } from 'vue';
import Present from './Present.vue';
import useSpeechRecognition from './useSpeechRecognition';

const { toggleListening, note } = useSpeechRecognition();

const basePath = import.meta.env.BASE_URL;
const startVideoSource = `${basePath}/password-cut.mp4`;
const endVideoSource = `${basePath}/end.mp4`;

const passwordCorrect = ref(false);
const shouldPlay = ref(false);
const secondVidFinished = ref(false)

const microphoneStatus = ref('idle');

const checkPassword = (result) => {
  const fuzzyMatch = ["kapoot", "kappa", "patric", "carpet", "draconus", "draconis"];

  if (fuzzyMatch.some(sub => result.includes(sub))) {
    passwordCorrect.value = true;
    toggleListening();
    this.$nextTick(() => {
      this.$refs.videoPlayer2.play().catch(error => {
        // Handle potential errors (e.g., if a user interaction is still required)
        console.error("Autoplay failed on preloaded video:", error);
      });
    });
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

watch(note, () => {
  console.log(note.value);
  checkPassword(note.value.toLowerCase())
});

</script>

<style scoped>
.video-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
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