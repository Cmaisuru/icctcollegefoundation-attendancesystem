<template>
  <header class="bg-[#006699] text-white py-4 sm:py-6 shadow-md relative font-poppins">
    <div class="container mx-auto px-4 sm:px-6 flex items-center justify-between">
      <h1 class="text-xl sm:text-2xl md:text-3xl font-bold tracking-wide whitespace-normal text-left sm:text-center flex-1">
        COLLEGE ATTENDANCE MANAGEMENT SYSTEM
      </h1>

      <!-- Info Button -->
      <button @click="showInfoPopup = true" aria-label="Information" class="ml-4 shrink-0">
        <svg class="w-7 h-7 sm:w-9 sm:h-9 text-white animate-pulse" fill="none" stroke="currentColor" stroke-width="2"
          viewBox="0 0 24 24" stroke-linecap="round" stroke-linejoin="round">
          <circle cx="12" cy="12" r="10" />
          <line x1="12" y1="16" x2="12" y2="12" />
          <circle cx="12" cy="8" r="1" fill="currentColor" />
        </svg>
      </button>
    </div>
  </header>

  <!-- Background -->
  <div class="min-h-screen flex items-start justify-center overflow-hidden"
    style="background: url('../assets/icct-colleges-rizal-philippines.jpg') no-repeat center center fixed; background-size: cover;">
    <div class="flex flex-col bg-white shadow-2xl rounded-3xl border border-[#b3e0f2] max-w-5xl w-full animate-fade-in overflow-hidden mt-10">

      <!-- Popup (Login Feedback) -->
      <div v-if="showPopup"
        :class="['fixed top-6 right-6 px-6 py-4 rounded-xl shadow-lg z-50 transition-all duration-300', popupType === 'success' ? 'bg-green-500 text-white' : 'bg-red-500 text-white']">
        {{ popupMessage }}
      </div>

      <!-- Header -->
      <div class="bg-[#006699] px-10 py-6 border-b border-[#005577] text-center" style="height: 25%;">
        <h1 class="text-3xl font-extrabold text-[#ffffff] tracking-wide drop-shadow-md text-center mb-2">
          ICCT CAINTA CAMPUS PORTAL
        </h1>
        <blockquote class="text-[#cce6ff] italic text-base md:text-lg leading-relaxed font-serif max-w-4xl mx-auto">
          “Education is the passport to the future, for tomorrow belongs to those who prepare for it today.”
        </blockquote>
      </div>

      <!-- Login Flex -->
      <div class="flex flex-col md:flex-row" style="height: 75%;">
        <!-- Instructor Login -->
        <div class="w-full md:w-1/2 p-10 overflow-auto">
          <form @submit.prevent="login" class="space-y-8">
            <div>
              <h1 class="text-3xl font-extrabold text-[#336699] tracking-wide drop-shadow-md text-center mb-8">
                INSTRUCTOR
              </h1>
              <label for="username" class="block mb-3 font-semibold text-gray-700 tracking-wide">Username:</label>
              <input id="username" v-model="username" required placeholder="Enter your username"
                class="w-full px-5 py-4 rounded-xl border border-gray-300 text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-4 focus:ring-[#66ccff] focus:border-[#006699] transition duration-300 shadow-sm" />
            </div>
            <div>
              <label for="password" class="block mb-3 font-semibold text-gray-700 tracking-wide">Password:</label>
              <input id="password" type="password" v-model="password" required placeholder="Enter your password"
                class="w-full px-5 py-4 rounded-xl border border-gray-300 text-gray-900 placeholder-gray-400 focus:outline-none focus:ring-4 focus:ring-[#66ccff] focus:border-[#006699] transition duration-300 shadow-sm" />
            </div>
            <button type="submit"
              class="w-full py-4 rounded-xl bg-[#006699] text-white font-bold text-lg hover:bg-[#005577] active:bg-[#004466] shadow-lg transition-all duration-300">
              Login
            </button>
          </form>
        </div>

        <!-- Right Panel -->
        <div class="w-full md:w-1/2 flex flex-col items-center justify-center p-10 bg-[#f0fbff] border-l border-[#b3e0f2] overflow-auto">
          <svg class="w-32 h-32 text-[#0099cc] animate-pulse-slow mb-8 drop-shadow-md" fill="none" stroke="currentColor"
            stroke-width="1.5" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round"
              d="M12 11c0-2.21 1.79-4 4-4s4 1.79 4 4a4 4 0 01-4 4c-1.1 0-2 .9-2 2m-4 0a4 4 0 014-4c2.21 0 4-1.79 4-4S16.21 3 14 3a4 4 0 00-4 4c0 1.1-.9 2-2 2m0 4a4 4 0 014 4c0 2.21-1.79 4-4 4S4 19.21 4 17" />
          </svg>
          <p class="text-[#004466] text-center text-lg font-medium max-w-md leading-relaxed">
            Welcome ICCT Student! Please verify your attendance through secure and seamless fingerprint access.
          </p>
        </div>
      </div>
    </div>
  </div>

  <!-- Info Popup Modal -->
  <div
  v-if="showInfoPopup"
  class="fixed inset-0 flex items-center justify-center z-50"
>
  <div
    class="bg-white w-11/12 max-w-lg min-h-[400px] rounded-2xl shadow-2xl p-8 text-center relative animate-bounce-in"
  >
    <button
      @click="showInfoPopup = false"
      aria-label="Close info popup"
      class="absolute top-4 right-4 text-gray-600 hover:text-gray-900 focus:outline-none focus:ring-2 focus:ring-blue-500 rounded-full p-3 text-2xl"
    >
      &times;
    </button>

    <h2 class="text-3xl font-semibold mb-6 text-[#006699]">About This App</h2>
    <p class="text-gray-700 mb-6 leading-relaxed">
      This College Attendance Management System was developed by the ICCT team to streamline attendance tracking and enhance student verification through fingerprint access.
    </p>
      <p class="text-gray-700 mb-6 leading-relaxed">
      Project Team:<br>
      Member 1<br>
      Member 2<br>
      Member 3<br>
      Member 4
      </p>
    <p class="text-gray-600 text-sm">
      © 2025 ICCT. All rights reserved.
    </p>
  </div>
</div>

</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, get } from 'firebase/database'

const firebaseConfig = {
  apiKey: "AIzaSyB9VPCKfkTzEZR04BP5NZegF1cLfz4-dE4",
  authDomain: "icct-attendancedb.firebaseapp.com",
  projectId: "icct-attendancedb",
  storageBucket: "icct-attendancedb.appspot.com",
  messagingSenderId: "575248656738",
  appId: "1:575248656738:web:097d086fca25a35e61e43b"
}

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
const database = getDatabase(app)

// State
const username = ref('')
const password = ref('')
const showPopup = ref(false)
const popupMessage = ref('')
const popupType = ref('success')
const showInfoPopup = ref(false)
const router = useRouter()

// Login
async function login() {
  if (!username.value || !password.value) {
    showError('Please enter username and password.')
    return
  }

  try {
    const snapshot = await get(dbRef(database, 'teachers'))

    if (!snapshot.exists()) {
      showError('No teacher records found.')
      return
    }

    const teachers = snapshot.val()
    const inputUsername = username.value.trim()
    const inputPassword = password.value.trim()

    const match = Object.entries(teachers).find(([_, data]) =>
      data?.username === inputUsername && data?.password === inputPassword
    )

    if (match) {
      const [_, data] = match
      localStorage.setItem('loggedInTeacher', data.username)
      showSuccess(`Welcome, ${data.username}!`)
      setTimeout(() => {
        showPopup.value = false
        router.push('/home')
      }, 2000)
    } else {
      showError('Invalid username or password.')
    }
  } catch (err) {
    console.error('Firebase error:', err)
    showError('Login failed. Try again later.')
  }
}

// Popup helpers
function showSuccess(message) {
  popupMessage.value = message
  popupType.value = 'success'
  showPopup.value = true
}
function showError(message) {
  popupMessage.value = message
  popupType.value = 'error'
  showPopup.value = true
  setTimeout(() => (showPopup.value = false), 2500)
}
</script>

<style scoped>
@keyframes pulse-slow {
  0%, 100% {
    transform: scale(1);
    opacity: 0.8;
  }
  50% {
    transform: scale(1.1);
    opacity: 1;
  }
}
.animate-pulse-slow {
  animation: pulse-slow 2.5s ease-in-out infinite;
}
.fixed {
  animation: fadeInUp 0.4s ease-out;
}
@keyframes fadeInUp {
  from {
    transform: translateY(20px);
    opacity: 0;
  }
  to {
    transform: translateY(0);
    opacity: 1;
  }
}
@keyframes bounce-in {
  0% {
    transform: scale(0.5);
    opacity: 0;
  }
  60% {
    transform: scale(1.2);
    opacity: 1;
  }
  80% {
    transform: scale(0.95);
  }
  100% {
    transform: scale(1);
  }
}

.animate-bounce-in {
  animation: bounce-in 0.6s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}

* {
  font-family: 'Poppins', sans-serif;
}
</style>
