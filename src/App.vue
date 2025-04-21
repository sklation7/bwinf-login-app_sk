<script setup>
import { ref, reactive } from 'vue';

let isLoggedIn = ref(false);
let currentPuzzle = ref(1);
let username = ref('');
let puzzleWord = ref('');
let userSolution = ref('');
let feedbackMessage = ref('');
let feedbackClass = ref('');
let explanation = ref('');
let circleVisible = ref(false);
let circlePosition = reactive({ top: '50%', left: '50%' });
let timer = null;
let missCount = ref(0); // Counter for missed clicks

// Gemeine Nachrichten für falsche Versuche im ersten Puzzle
const wordPuzzleRoasts = [
  "Hast du überhaupt gelesen, was du machen sollst?",
  "Das ist doch nicht so schwer, oder?",
  "Bist du sicher, dass du das ernst meinst?",
  "Ein Kind könnte das lösen, und du nicht?",
  "Das ist peinlich. Versuch's nochmal.",
  "Vielleicht solltest du das Alphabet nochmal lernen.",
  "Das ist wirklich traurig anzusehen.",
  "Willst du absichtlich verlieren?",
  "Das ist ja schlimmer als ich dachte.",
  "Du machst das absichtlich falsch, oder?",
];

// Gemeine Nachrichten für das Reflex-Puzzle
const reflexPuzzleRoasts = [
  "Die blaue Kugel ist direkt vor dir! Bist du blind?",
  "Vielleicht sind Reflexe einfach nicht dein Ding.",
  "Schon wieder daneben? Ernsthaft?",
  "Weißt du überhaupt, was 'schnell' bedeutet?",
  "Die Kugel lacht dich gerade aus.",
  "Das ist ja schlimmer als ein Faultier.",
  "Du bist langsamer als meine Großmutter.",
  "Das ist ja fast schon Kunst, so schlecht zu sein.",
  "Hast du überhaupt versucht zu klicken?",
  "Vielleicht solltest du aufgeben.",
];

function getRandomRoast(roastArray) {
  return roastArray[Math.floor(Math.random() * roastArray.length)];
}

function resetToLoginPage() {
  isLoggedIn.value = false;
  currentPuzzle.value = 1; // Reset to the first puzzle
  feedbackMessage.value = '';
  feedbackClass.value = '';
  puzzleWord.value = '';
  userSolution.value = '';
  missCount.value = 0; // Reset miss count
}

function generatePuzzleWord() {
  const originalWord = 'apfel';
  explanation.value = `Das Wort "${originalWord}" wurde um 2 Positionen im Alphabet verschoben. Bitte gib das verschobene Wort ein.`;
  puzzleWord.value = originalWord
      .split('')
      .map((char) => {
        if (char >= 'a' && char <= 'z') {
          return String.fromCharCode(((char.charCodeAt(0) - 97 + 2) % 26) + 97);
        }
        return char;
      })
      .join('');
  feedbackMessage.value = '';
  feedbackClass.value = '';
}

function checkSolution() {
  if (userSolution.value.toLowerCase() === "asd") {
    // Secret solution bypasses the puzzle
    feedbackMessage.value = 'Wow, du hast das geheime Passwort gefunden! Weiter zum nächsten Puzzle...';
    feedbackClass.value = 'text-green-500';
    setTimeout(() => {
      currentPuzzle.value = 2; // Move to the second puzzle
      startCirclePuzzle();
    }, 1000);
  } else if (userSolution.value.toLowerCase() === puzzleWord.value) {
    // Regular solution
    feedbackMessage.value = 'Richtig! Weiter zum nächsten Puzzle...';
    feedbackClass.value = 'text-green-500';
    setTimeout(() => {
      currentPuzzle.value = 2; // Move to the second puzzle
      startCirclePuzzle();
    }, 1000);
  } else {
    // Incorrect solution
    feedbackMessage.value = getRandomRoast(wordPuzzleRoasts);
    feedbackClass.value = 'text-red-500';
  }
}

function startCirclePuzzle() {
  explanation.value = 'Klicke die blaue Kugel so schnell wie möglich an! Wenn du daneben klickst, erscheint sie nach 1 Sekunde an einer zufälligen Position.';
  spawnCircle();
}

function spawnCircle() {
  circleVisible.value = true;
  circlePosition.top = `${Math.random() * 80 + 10}%`;
  circlePosition.left = `${Math.random() * 80 + 10}%`;
  clearTimeout(timer);
  timer = setTimeout(() => {
    circleVisible.value = false;
    missCount.value++; // Increment miss count
    if (missCount.value >= 3) {
      feedbackMessage.value = getRandomRoast(reflexPuzzleRoasts);
      feedbackClass.value = 'text-red-500';
    } else {
      feedbackMessage.value = 'Daneben! Versuch es nochmal.';
      feedbackClass.value = 'text-yellow-500';
    }
    setTimeout(spawnCircle, 1000);
  }, 800);
}

function handleCircleClick() {
  clearTimeout(timer); // Stop the timer for the current circle
  circleVisible.value = false; // Hide the circle
  feedbackMessage.value = 'Tolle Reflexe! Du hast das Puzzle abgeschlossen!';
  feedbackClass.value = 'text-green-500'; // Success feedback
  missCount.value = 0; // Reset miss count on success
  setTimeout(() => {
    isLoggedIn.value = true; // Log the user in after a short delay
  }, 1000);
}
</script>

<template>
  <div class="min-h-screen bg-gray-900 text-gray-100 flex items-center justify-center relative">
    <!-- Puzzle 1 -->
    <div v-if="!isLoggedIn && currentPuzzle === 1" class="bg-gray-800 shadow-lg rounded-lg p-8 w-96">
      <h1 class="text-2xl font-bold text-center text-gray-100 mb-6">
        Log in
      </h1>
      <div class="flex flex-col">
        <input
            v-model="username"
            type="text"
            placeholder="Benutzername"
            class="p-2 mb-4 border border-gray-600 rounded-lg bg-gray-700 text-gray-100 focus:outline-none focus:ring-2 focus:ring-blue-500"
            @keyup.enter="generatePuzzleWord"
        />
        <button
            @click="generatePuzzleWord"
            class="w-full bg-blue-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-blue-600 transition duration-300"
        >
          Puzzle lösen
        </button>
        <div v-if="puzzleWord" class="mt-6">
          <p class="text-sm text-gray-300 mb-4">{{ explanation }}</p>
          <input
              v-model="userSolution"
              type="text"
              placeholder="Deine Lösung"
              class="p-2 mb-4 border border-gray-600 rounded-lg bg-gray-700 text-gray-100 focus:outline-none focus:ring-2 focus:ring-green-500 w-full"
              @keyup.enter="checkSolution"
          />
          <button
              @click="checkSolution"
              class="w-full bg-green-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-green-600 transition duration-300"
          >
            Lösung einreichen
          </button>
          <p :class="['mt-4 text-center', feedbackClass]">
            {{ feedbackMessage }}
          </p>
        </div>
      </div>
    </div>

    <!-- Puzzle 2 -->
    <div v-if="!isLoggedIn && currentPuzzle === 2" class="bg-gray-800 shadow-lg rounded-lg p-8 w-96">
      <h1 class="text-2xl font-bold text-center text-gray-100 mb-6">
        Puzzle 2: Reflex-Herausforderung
      </h1>
      <p class="text-sm text-gray-300 mb-4 text-center">{{ explanation }}</p>
      <p :class="['mt-4 text-center', feedbackClass]">
        {{ feedbackMessage }}
      </p>
      <div
          v-if="circleVisible"
          :style="{ top: circlePosition.top, left: circlePosition.left }"
          class="absolute w-8 h-8 bg-blue-500 rounded-full cursor-pointer"
          @click="handleCircleClick"
      ></div>
    </div>

    <!-- Logged In -->
    <div v-if="isLoggedIn" class="bg-gray-800 shadow-lg rounded-lg p-8 w-96 text-center">
      <div class="absolute top-4 left-4 text-sm font-bold text-gray-400">
        Willkommen, {{ username }}!
      </div>
      <button
          @click="resetToLoginPage"
          class="w-full bg-red-500 text-white font-bold py-2 px-4 rounded-lg hover:bg-red-600 transition duration-300 mt-4"
      >
        Zurück
      </button>
    </div>
  </div>
</template>