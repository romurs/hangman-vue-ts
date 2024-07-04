<script setup lang="ts">
import GameHeader from './components/GameHeader.vue'
import GameFigure from './components/GameFigure.vue'
import GameWrongLetters from './components/GameWrongLetters.vue'
import GameWord from './components/GameWord.vue'
import GameFinalMessage from './components/GameFInalMessage.vue'
import GameNotification from './components/GameNotification.vue'
import { ref, computed, watch } from 'vue'
import axios from 'axios'

const word = ref('')
const randomWord = async () => {
  try {
    const { data } = await axios<{ FirstName: string }>(
      'https://api.randomdatatools.ru/?unescaped=false&params=FirstName'
    )
    word.value = data.FirstName.toLowerCase()
  } catch (err) {
    console.log(err)
    word.value = ""
  }
}
randomWord()

const letters = ref<string[]>([])
const correctLetters = computed(() => letters.value.filter((x) => word.value.includes(x)))
const wrongLetters = computed(() => letters.value.filter((x) => !word.value.includes(x)))
const notification = ref<InstanceType<typeof GameNotification> | null>(null)
const popup = ref<InstanceType<typeof GameFinalMessage> | null>(null)

const isWin = computed(() => [...word.value].every((x) => correctLetters.value.includes(x)))
const isLose = computed(() => wrongLetters.value.length === 6)
watch(correctLetters, () => {
  if (isWin.value) {
    popup.value?.open('win')
  }
})
watch(wrongLetters, () => {
  if (isLose.value) {
    popup.value?.open('lose')
  }
})

window.addEventListener('keydown', ({ key }) => {
  if (isWin.value || isLose.value) return

  if (letters.value.includes(key)) {
    notification.value?.open()
    setTimeout(() => notification.value?.close(), 2000)
    return
  }

  if (/[а-яА-ЯёЁ]/.test(key)) {
    letters.value.push(key.toLowerCase())
  }
})

const restart = () => {
  randomWord()
  letters.value = []
  popup.value?.close()
}
</script>

<template>
  <div id="app">
    <GameHeader />
    <div class="game-container">
      <GameFigure :count-wrong-letters="wrongLetters.length" />
      <GameWrongLetters :wrongLetters="wrongLetters" />
      <GameWord :word="word" :correctLetters="correctLetters" />
    </div>

    <GameFinalMessage ref="popup" :word="word" @restart="restart" />
    <GameNotification ref="notification" />
  </div>
</template>

<style scoped></style>
