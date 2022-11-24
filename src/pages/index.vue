<script setup lang="ts">
const user = useUserStore()
const name = $ref(user.savedName)

const voices = ref<SpeechSynthesisVoice[]>([])
const voice = ref<SpeechSynthesisVoice | undefined>(undefined)
const texts = ref<string[]>([''])
const text = computed(() => texts.value[texts.value.length - 1])

const speechRecognition = useSpeechRecognition({
  lang: 'fr-FR',
  continuous: true,
})
console.log(texts.value)
const speech = useSpeechSynthesis(text, {voice})

onMounted(() => {
  if (speech.isSupported.value && speechRecognition.isSupported.value) {
    // load at last
    setTimeout(() => {
      const synth = window.speechSynthesis
      voices.value = synth.getVoices()
      voice.value = voices.value[0]
    })
  }
})

function listenSpeech() {
  speechRecognition.start()
}

const playSpeech = () => {
  speech.speak()
}

function stopSpeech() {
  if (speechRecognition.isListening.value) {
    texts.value.push(speechRecognition.result.value)
    speechRecognition.result.value = ''
    speechRecognition.stop()
    playSpeech()
  }
}
</script>

<template>
  <div flex flex-col justify-center>
    <select v-model="voice">
      <option bg="$vp-c-bg" disabled>
        Select Language
      </option>
      <option
        v-for="(voice, i) in voices"
        :key="i"
        bg="$vp-c-bg"
        :value="voice"
      >
        {{ `${voice.name} (${voice.lang})` }}
      </option>
    </select>
    <button v-if="!speechRecognition.isListening.value" @click="listenSpeech">
      Listen
    </button>
    <button v-else @click="stopSpeech">
      Stop
    </button>
    <textarea v-for="text in texts" :value="text"/>
  </div>
</template>

<route lang="yaml">
meta:
  layout: home
</route>
