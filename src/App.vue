<template>
  <form @submit.prevent="submitForm">
    <input
      id="date"
      type="text"
      :placeholder="formattedDate(new Date())"
      v-model="inputDate"
      v-mask="'####-##-##'"
    >
    <button type="submit">Установить дату</button>
  </form>
  <Calendar :date="date" :key="key" @date-selected="gotSelectedDate" />
</template>

<script setup lang="ts">
import { ref } from 'vue'
import Calendar from './components/Calendar.vue'
import { formattedDate } from './helpers'

let key = 0
const inputDate = ref('')
const date = ref()

function submitForm () {
  if (isNaN(new Date(inputDate.value).getTime())) return
  date.value = inputDate.value
  key++
}

function gotSelectedDate(date: Date) {
  console.log(`Выбранная дата: ${formattedDate(date)}`)
}
</script>

<style scoped>
form, input, button {
  font-family:Verdana, Geneva, Tahoma, sans-serif;
}
button {
  cursor: pointer;
}
</style>
