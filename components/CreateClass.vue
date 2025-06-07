<template>
  <main class="min-h-screen flex items-center justify-center -mt-25 pt-25 px-4 font-poppins">
    <div class="mw-full max-w-4xl mx-auto bg-white rounded shadow">
      <img src="../assets/bg-3.png" alt="Form Header" class="w-full h-40" />

      <div class="p-6">
        <h2 class="text-3xl font-bold mb-3 text-[#006699] text-center">CREATE NEW CLASS</h2>
        <p class="text-gray-700 text-center">
          New Semester? Lets recreate a new class schedule for our colleagues and students!
        </p>
        <p class="mb-8 text-gray-700 text-center">
          Note: Incase of errors, you can delete the course and section under "View Classes" section.
        </p>

        <form @submit.prevent="handleSubmit" class="space-y-4">
          <div>
            <label class="block mb-1 font-semibold" for="course">Course Name</label>
            <input
              id="course"
              v-model="course"
              type="text"
              required
              placeholder="Enter course name (e.g., Mathematics)"
              class="w-full border border-gray-300 rounded px-3 py-2"
            />
          </div>

          <div>
            <label class="block mb-1 font-semibold" for="section">Section</label>
            <input
              id="section"
              v-model="section"
              type="text"
              required
              placeholder="Enter section name (e.g., A)"
              class="w-full border border-gray-300 rounded px-3 py-2"
            />
          </div>

          <div>
            <label class="block mb-1 font-semibold" for="startTime">Start Time</label>
            <input
              id="startTime"
              v-model="startTime"
              type="time"
              required
              class="w-full border border-gray-300 rounded px-3 py-2"
            />
          </div>

          <div>
            <label class="block mb-1 font-semibold" for="endTime">End Time</label>
            <input
              id="endTime"
              v-model="endTime"
              type="time"
              required
              class="w-full border border-gray-300 rounded px-3 py-2"
            />
          </div>

          <button
            type="submit"
            class="w-full bg-[#006699] text-white py-2 rounded hover:bg-[#004466] transition"
            :disabled="loading"
          >
            {{ loading ? "Creating..." : "Create Class" }}
          </button>
        </form>
      </div>

      <!-- Old message display removed, replaced by animated popup below -->

      <!-- Animated popup message -->
      <transition name="fade-slide">
        <div
          v-if="message"
          :class="['fixed bottom-8 left-1/2 transform -translate-x-1/2 px-6 py-3 rounded shadow-lg font-semibold text-white', popupClass]"
          @click="clearMessage"
          role="alert"
          tabindex="0"
        >
          {{ message }}
        </div>
      </transition>
    </div>
  </main>
</template>

<script setup>
import { ref, watch } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, set, get } from 'firebase/database'

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

const course = ref('')
const section = ref('')
const startTime = ref('')
const endTime = ref('')
const loading = ref(false)
const message = ref('')
const messageClass = ref('')

// popupClass is for bg color in the popup (green or red)
const popupClass = ref('bg-green-600')

// Clear message manually on click
function clearMessage() {
  message.value = ''
}

function convertToAMPM(time24) {
  const [hourStr, minute] = time24.split(':')
  let hour = parseInt(hourStr)
  const ampm = hour >= 12 ? 'PM' : 'AM'
  hour = hour % 12 || 12
  return `${hour}:${minute} ${ampm}`
}

async function handleSubmit() {
  if (!course.value || !section.value || !startTime.value || !endTime.value) {
    message.value = 'Please fill in all fields.'
    messageClass.value = 'text-red-600'
    popupClass.value = 'bg-red-600'
    return
  }

  loading.value = true
  message.value = ''

  try {
    const courseRef = dbRef(database, `courses/${course.value}`)
    const snapshot = await get(courseRef)

    if (snapshot.exists()) {
      const existingData = snapshot.val()

      // 🚫 Check if section already exists under this course
      if (existingData.sections && existingData.sections[section.value]) {
        message.value = 'This course and section already exists.'
        messageClass.value = 'text-red-600'
        popupClass.value = 'bg-red-600'
        loading.value = false
        return
      }

      // ✅ Update course with the new section
      const updatedData = {
        name: course.value,
        sections: {
          ...(existingData.sections || {}),
          [section.value]: {
            startTime: convertToAMPM(startTime.value),
            endTime: convertToAMPM(endTime.value)
          }
        }
      }

      await set(courseRef, updatedData)
    } else {
      // ✅ New course entry
      await set(courseRef, {
        name: course.value,
        sections: {
          [section.value]: {
            startTime: convertToAMPM(startTime.value),
            endTime: convertToAMPM(endTime.value)
          }
        }
      })
    }

    message.value = 'Class created successfully!'
    messageClass.value = 'text-green-600'
    popupClass.value = 'bg-green-600'

    // Clear form fields
    course.value = ''
    section.value = ''
    startTime.value = ''
    endTime.value = ''
  } catch (error) {
    console.error(error)
    message.value = 'Failed to create class.'
    messageClass.value = 'text-red-600'
    popupClass.value = 'bg-red-600'
  } finally {
    loading.value = false
  }
}


// Auto clear message after 3.5 seconds
watch(message, (val) => {
  if (val) {
    setTimeout(() => {
      message.value = ''
    }, 3500)
  }
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

* {
  font-family: 'Poppins', sans-serif;
}

/* Transition for popup */
.fade-slide-enter-active,
.fade-slide-leave-active {
  transition: opacity 0.4s ease, transform 0.4s ease;
}
.fade-slide-enter-from,
.fade-slide-leave-to {
  opacity: 0;
  transform: translateY(20px);
}
.fade-slide-enter-to,
.fade-slide-leave-from {
  opacity: 1;
  transform: translateY(0);
}

/* Popup styling */
.fixed {
  z-index: 9999;
  cursor: pointer;
  user-select: none;
}
</style>
