<template>
  <div class="calendar">
    <div class="calendar__control">
        <button class="calendar__control-prev" @click="changeMonth('prev')">
            <svg viewBox="0 0 512 512" width="24px" height="24px" xmlns="http://www.w3.org/2000/svg"><title/><g data-name="1" id="_1"><path d="M353,450a15,15,0,0,1-10.61-4.39L157.5,260.71a15,15,0,0,1,0-21.21L342.39,54.6a15,15,0,1,1,21.22,21.21L189.32,250.1,363.61,424.39A15,15,0,0,1,353,450Z"/></g></svg>
        </button>
        <h3 class="calendar__info">{{ monthYear }}</h3>
        <button class="calendar__control-next" @click="changeMonth">
            <svg viewBox="0 0 512 512" width="24px" height="24px" xmlns="http://www.w3.org/2000/svg"><title/><g data-name="1" id="_1"><path d="M202.1,450a15,15,0,0,1-10.6-25.61L365.79,250.1,191.5,75.81A15,15,0,0,1,212.71,54.6l184.9,184.9a15,15,0,0,1,0,21.21l-184.9,184.9A15,15,0,0,1,202.1,450Z"/></g></svg>
        </button>
    </div>
    <div class="calendar__week">
        <div
            v-for="wd in weekdays"
            :key="wd"
            class="calendar__weekday"
        >
        {{ wd }}
        </div>
    </div>
    <div class="calendar__days">
        <div
            v-for="d in days"
            :key="d"
            class="calendar__day"
            :class="[
                {'calendar__day--actual-month': d.getMonth() === date.getMonth()},
                {'calendar__day--selected-day': isSameDate(d, date)},
                {'calendar__day--current-day': isSameDate(d, new Date())},
            ]"
            @click="selectDate(d)"
        >
        {{ d.getDate() }}
        </div>
    </div>
    <div class="calendar__lang">
        <div class="calendar__icon">
            <svg viewBox="0 0 512 512" width="28px" height="28px" xmlns="http://www.w3.org/2000/svg"><title/><g data-name="1" id="_1"><path d="M160.91,411.51a15,15,0,0,1-15-15v-295A53.56,53.56,0,0,1,199.41,48H423.5a15,15,0,0,1,15,15V358a15,15,0,0,1-30,0V78H199.41a23.52,23.52,0,0,0-23.5,23.49v295A15,15,0,0,1,160.91,411.51Z"/><path d="M423.5,450H199.41a53.5,53.5,0,1,1,0-107H423.5a15,15,0,0,1,0,30H199.41a23.5,23.5,0,1,0,0,47H423.5a15,15,0,0,1,0,30Z"/><path d="M406.14,447.58a15,15,0,0,1-15-15V358a15,15,0,0,1,30,0v74.57A15,15,0,0,1,406.14,447.58Z"/><path d="M334.2,298.26a15,15,0,0,1-14.41-10.88l-27.58-96.53-27.59,96.53a15,15,0,1,1-28.84-8.24l42-147a15,15,0,0,1,28.85,0l42,147a15,15,0,0,1-10.3,18.54A14.71,14.71,0,0,1,334.2,298.26Z"/><path d="M323.57,258.24H261.64a15,15,0,0,1,0-30h61.93a15,15,0,0,1,0,30Z"/></g></svg>
        </div>
        <select class="calendar__select" v-model="lang">
            <option
                v-for="lang in langs"
                :key="lang"
                :value="lang"
            >
            {{ lang.nativeName ?? lang.name }}
            </option>
        </select>
    </div>
  </div>
</template>

<script setup lang="ts">
import { formattedDate, isSameDate } from '../helpers'
import { computed, defineProps, onBeforeMount, ref } from 'vue'
import ISO6391 from 'iso-639-1'

const emit = defineEmits<{
  dateSelected: [date: Date]
}>()

const props = defineProps({date: {
    type: String,
    default: formattedDate(new Date)
}})

let langs = []
const lang = ref(ISO6391.getLanguages(['ru'])[0])
const date = ref(new Date(props.date))

onBeforeMount(() => {
    langs = ISO6391.getLanguages(ISO6391.getAllCodes())
    const availableLangs = Intl.DateTimeFormat.supportedLocalesOf(langs.map(lang => lang.code))
    langs = ISO6391.getLanguages(availableLangs).filter(l => l.name !== '')
})

const weekdays = computed (() => {
    const day = new Date('2025-08-18')
    const week = []
    for (let i = 0; i < 7; i++) {
        week.push(new Intl.DateTimeFormat(lang.value.code, {
            weekday: "short"
        }).format(day))
        day.setDate(day.getDate() + 1)
    }
    return week
})

const days = computed(() => {
    const days = []

    const firstMonthDay = new Date(date.value.getFullYear(), date.value.getMonth(), 1)
    let weekday = firstMonthDay.getDay() === 0 ? 7 : firstMonthDay.getDay()
    const firstDay = new Date(firstMonthDay)
    firstDay.setDate(firstDay.getDate() - weekday + 1)

    const lastMonthDay = new Date(date.value.getFullYear(), date.value.getMonth() + 1, 0)
    weekday = lastMonthDay.getDay() === 0 ? 7 : lastMonthDay.getDay()
    const lastDay = new Date(lastMonthDay)
    lastDay.setDate(lastDay.getDate() + (7 - weekday))

    const currentDay = firstDay

    while (currentDay.getTime() <= lastDay.getTime()) {
        days.push(new Date(currentDay))
        currentDay.setDate(currentDay.getDate() + 1)
    }

    return days
})

const monthYear = computed(() => {
    return `${date.value.getFullYear()} ${new Intl.DateTimeFormat(lang.value.code, {
            month: "short"
        }).format(date.value)}`
})

function changeMonth(direction?: string) {
    const shift = direction === 'prev' ? -1 : 1
    date.value = new Date(date.value.getFullYear(), date.value.getMonth() + shift, 1)
}

function selectDate(selectedDate: Date) {
    date.value = selectedDate
    emit('dateSelected', date.value)
}
</script>

<style scoped lang="scss">
.calendar {
    width: 350px;
    font-family: Verdana, Geneva, Tahoma, sans-serif;
    background-color: white;

    &__control {
        display: flex;
        gap: 4px;
        padding: 4px 0;

        &-prev, &-next {
            all: unset;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
        }
    }

    &__info {
        display: flex;
        flex: 1;
        justify-content: center;
        align-items: center;
    }

    &__week {
        display: flex;
    }

    &__weekday,
    &__day {
        display: flex;
        width: 50px;
        height: 50px;
        justify-content: center;
        align-items: center;
    }

    &__day {
        color: rgb(173, 173, 173);
        border-radius: 4px;
        box-sizing: border-box;
        cursor: pointer;
        transition: background-color .3s ease;

        &:hover {
            background-color: rgb(243, 243, 243);
        }

        &--actual-month {
            color: black;
        }

        &--selected-day {
            border: 2px rgb(124, 96, 201) solid;
        }

        &--current-day {
            background-color: rgb(151, 137, 194);
            color: white;
        }
    }

    &__days {
        display: flex;
        flex-wrap: wrap;
    }

    &__lang {
        display: flex;
        justify-content: center;
        align-items: center;
    }

    &__select {
        display: flex;
        flex: 1;
        gap: 4px;
        padding: 4px 0;
        font-family: inherit;
        cursor: pointer;
    }

    &__icon {
        display: flex;
        justify-content: center;
        align-items: center;
    }
}
</style>