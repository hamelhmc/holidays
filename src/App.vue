<template>
  <div class="h-screen w-screen bg-cover bg-center flex flex-col items-center justify-center text-center p-4"
    :style="{ backgroundColor: backgroundColor }">
    <button @click="toggleSettings"
      class="absolute top-4 left-4 p-2 bg-gray-800 text-white rounded-full shadow transform transition hover:scale-110">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor"
        class="w-6 h-6">
        <path stroke-linecap="round" stroke-linejoin="round"
          d="M12 8.25v7.5m0-7.5h.008v7.5H12m9-3.75a9 9 0 11-18 0 9 9 0 0118 0z" />
      </svg>
    </button>

    <h1 class="text-6xl md:text-8xl font-bold text-white drop-shadow-md animate-pulse font-japanese">{{ title }}
    </h1>
    <div class="text-9xl mb-6 animate-bounce-slow">{{ emoji }}</div>
    <div class="flex gap-4 text-gray-200 text-4xl md:text-6xl animate-fade-in">
      <div class="flex flex-col items-center">
        <span class="animate-count text-white">{{ countdown.days }}</span>
        <span class="text-sm md:text-lg">days</span>
      </div>
      <div class="flex flex-col items-center">
        <span class="animate-count text-white">{{ countdown.hours }}</span>
        <span class="text-sm md:text-lg">hours</span>
      </div>
      <div class="flex flex-col items-center">
        <span class="animate-count text-white">{{ countdown.minutes }}</span>
        <span class="text-sm md:text-lg">min</span>
      </div>
      <div class="flex flex-col items-center">
        <span class="animate-count text-white">{{ countdown.seconds }}</span>
        <span class="text-sm md:text-lg">sec</span>
      </div>
    </div>

    <div v-if="showSettings"
      class="absolute top-0 left-0 h-full w-full bg-black bg-opacity-50 flex items-center justify-center">
      <div class="bg-white p-6 rounded shadow-lg w-80">
        <h2 class="text-xl font-bold mb-4">Configuración</h2>

        <label class="block mb-2">
          <span class="text-gray-700">Título:</span>
          <input v-model="editableTitle" class="mt-1 block w-full p-2 border rounded" />
        </label>

        <label class="block mb-2">
          <span class="text-gray-700">Fecha objetivo:</span>
          <input type="datetime-local" v-model="editableDate" class="mt-1 block w-full p-2 border rounded" />
        </label>

        <label class="block mb-2">
          <span class="text-gray-700">Emoji:</span>
          <input v-model="editableEmoji" class="mt-1 block w-full p-2 border rounded" />
        </label>

        <label class="block mb-4">
          <span class="text-gray-700">Color de fondo:</span>
          <input type="color" v-model="editableBackgroundColor" class="mt-1 block w-full p-2 border rounded" />
        </label>

        <div class="flex justify-between">
          <button @click="applySettings"
            class="px-4 py-2 bg-blue-500 text-white font-bold rounded shadow hover:bg-blue-600">Guardar</button>
          <button @click="toggleSettings"
            class="px-4 py-2 bg-red-500 text-white font-bold rounded shadow hover:bg-red-600">Cancelar</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onUnmounted, watch } from 'vue';

// Import custom font for Japanese style
import "@fontsource/noto-sans-jp";

// Load settings from localStorage
const savedSettings = JSON.parse(localStorage.getItem('countdownSettings') || '{}');

const targetDate = ref(new Date(savedSettings.targetDate || '2025-08-01T00:00:00'));
const title = ref(savedSettings.title || 'JAPÓN');
const emoji = ref(savedSettings.emoji || '🎉');
const backgroundColor = ref(savedSettings.backgroundColor || '#F8E2CF');

const editableDate = ref(targetDate.value.toISOString().slice(0, 16));
const editableTitle = ref(title.value);
const editableEmoji = ref(emoji.value);
const editableBackgroundColor = ref(backgroundColor.value);

const showSettings = ref(false);

const countdown = ref({
  days: 0,
  hours: 0,
  minutes: 0,
  seconds: 0,
});

const updateCountdown = () => {
  const now = new Date();
  const difference = targetDate.value.getTime() - now.getTime();

  if (difference <= 0) {
    countdown.value = { days: 0, hours: 0, minutes: 0, seconds: 0 };
    return;
  }

  countdown.value.days = Math.floor(difference / (1000 * 60 * 60 * 24));
  countdown.value.hours = Math.floor((difference / (1000 * 60 * 60)) % 24);
  countdown.value.minutes = Math.floor((difference / (1000 * 60)) % 60);
  countdown.value.seconds = Math.floor((difference / 1000) % 60);
};

const toggleSettings = () => {
  showSettings.value = !showSettings.value;
};

const applySettings = () => {
  title.value = editableTitle.value;
  emoji.value = editableEmoji.value;
  backgroundColor.value = editableBackgroundColor.value;
  targetDate.value = new Date(editableDate.value);

  // Save settings to localStorage
  localStorage.setItem(
    'countdownSettings',
    JSON.stringify({
      title: title.value,
      emoji: emoji.value,
      backgroundColor: backgroundColor.value,
      targetDate: targetDate.value,
    })
  );

  toggleSettings();
};

const requestNotificationPermission = async () => {
  if (Notification.permission === 'default') {
    await Notification.requestPermission();
  }
};

const scheduleWeeklyNotifications = () => {
  if (Notification.permission !== 'granted') return;

  const now = new Date();
  const daysRemaining = Math.floor((targetDate.value.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));

  if (daysRemaining > 0) {
    new Notification(`Quedan ${daysRemaining} días para ${title.value}`, {
      body: `¡No olvides prepararte para el gran día!`,
      icon: 'https://via.placeholder.com/128',
    });
  }
};

watch(title, (newTitle) => {
  document.title = newTitle;
});

let intervalId;
let weeklyNotificationIntervalId;

onMounted(() => {
  updateCountdown();
  intervalId = setInterval(updateCountdown, 1000);

  requestNotificationPermission();
  scheduleWeeklyNotifications();
  weeklyNotificationIntervalId = setInterval(scheduleWeeklyNotifications, 7 * 24 * 60 * 60 * 1000);

  document.title = title.value;
});

onUnmounted(() => {
  clearInterval(intervalId);
  clearInterval(weeklyNotificationIntervalId);
});
</script>

<style scoped>
body {
  margin: 0;
  font-family: 'Noto Sans JP', sans-serif;
  background-color: #F8E2CF;
}

.font-japanese {
  font-family: 'Noto Sans JP', sans-serif;
}

.animate-bounce-slow {
  animation: bounce-slow 4s infinite;
}

@keyframes bounce-slow {

  0%,
  100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-5px);
  }
}

.animate-fade-in {
  animation: fade-in 2s ease-in;
}

@keyframes fade-in {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.animate-count {
  animation: pop 0.5s ease-out infinite alternate;
}

@keyframes pop {
  from {
    transform: scale(1);
  }

  to {
    transform: scale(1.1);
  }
}
</style>
