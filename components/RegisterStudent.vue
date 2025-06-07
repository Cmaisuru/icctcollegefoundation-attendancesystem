<template>
<main class="min-h-screen flex items-center justify-center -mb-15 -mt-15 font-poppins">
    <div class="w-full max-w-3xl bg-white rounded-xl shadow relative">
       <img src="../assets/bg-2.png" alt="Form Header" class="w-full h-40" />
      <!-- Popup message -->
      <div
        v-if="showPopup"
        :class="[
          'fixed top-6 right-6 px-6 py-4 rounded-xl shadow-lg z-50 transition-all duration-300',
          popupType === 'success' ? 'bg-green-500 text-white' : 'bg-red-500 text-white'
        ]"
      >
        {{ popupMessage }}
      </div>
<div class="p-6">
     <h2 class="text-3xl font-bold mb-3 text-[#006699] text-center">STUDENT REGISTRATION</h2>
    <p class="text-gray-700 text-center">
      Hello fellow instructors! Make sure to register your students information below.
    </p>
     <p class="mb-8 text-gray-700 text-center">
      Note: You can delete the student account in the "View All Students" if you made an error.
    </p>

    

      <form @submit.prevent="handleSubmit" class="space-y-6">
      <div>
        <label for="studentId" class="block mb-1 font-bold text-gray-700">Student ID:</label>
        <input
          id="studentId"
          type="text"
          v-model="studentId"
          disabled
          class="w-full px-4 py-2 border rounded-lg bg-gray-100 cursor-not-allowed"
        />
      </div>

      <div>
        <label for="name" class="block mb-1 font-bold text-gray-700">Student Name:</label>
        <input
          id="name"
          type="text"
          v-model="name"
          placeholder="Enter student name"
          required
          class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-[#006699]"
        />
      </div>

      <div>
        <label for="course" class="block mb-1 font-bold text-gray-700">Select Course:</label>
        <select
          id="course"
          v-model="selectedCourse"
          required
          class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-[#006699]"
        >
          <option value="" disabled>Select a course</option>
          <option
            v-for="(course, key) in courseList"
            :key="key"
            :value="key"
          >
            {{ course.name }}
          </option>
        </select>
      </div>

      <div v-if="selectedCourse">
        <label for="section" class="block mb-1 font-medium text-gray-700">Select Section:</label>
        <select
          id="section"
          v-model="selectedSection"
          required
          class="w-full px-4 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-[#006699]"
        >
          <option value="" disabled>Select a section</option>
          <option
            v-for="(sectionData, key) in availableSections"
            :key="key"
            :value="key"
          >
            {{ key }} ({{ sectionData.startTime }} - {{ sectionData.endTime }})
          </option>
        </select>
      </div>

      <button
        type="submit"
        :disabled="loading"
        class="w-full bg-[#006699] text-white py-3 rounded-lg font-semibold hover:bg-[#004466] transition"
      >
        {{ loading ? 'Registering...' : 'Register Student' }}
      </button>
    </form>
  </div>
  </div>
  </main>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, onValue, push, query, orderByChild, equalTo, get } from 'firebase/database'

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

// Reactive variables
const studentId = ref(generateRandomId())
const name = ref('')
const selectedCourse = ref('')
const selectedSection = ref('')
const loading = ref(false)

const courseList = ref({})

// Popup variables
const showPopup = ref(false)
const popupMessage = ref('')
const popupType = ref('success') // 'success' or 'error'

// Generate random 6-digit ID as string
function generateRandomId() {
  return Math.floor(100000 + Math.random() * 900000).toString()
}

// Load courses and their sections from Firebase realtime DB
onMounted(() => {
  const courseRef = dbRef(database, 'courses')
  onValue(courseRef, (snapshot) => {
    if (snapshot.exists()) {
      courseList.value = snapshot.val()
      console.log('Loaded courses:', courseList.value)  // Debug log
    } else {
      console.log('No courses data found.')
      courseList.value = {}
    }
  })
})

// Reset selectedSection when selectedCourse changes
watch(selectedCourse, () => {
  selectedSection.value = ''
})

// Compute available sections for the selected course safely
const availableSections = computed(() => {
  if (!selectedCourse.value) return {}
  const course = courseList.value[selectedCourse.value]
  if (!course || !course.sections) return {}
  return course.sections
})

// Show popup with message and type
function showPopupMessage(message, type = 'success') {
  popupMessage.value = message
  popupType.value = type
  showPopup.value = true

  setTimeout(() => {
    showPopup.value = false
  }, 3000)
}

// Handle form submission
async function handleSubmit() {
  if (!name.value || !selectedCourse.value || !selectedSection.value) {
    showPopupMessage('Please fill in all fields.', 'error')
    return
  }

  loading.value = true

  try {
    const studentsRef = dbRef(database, 'students')

    // Check if student name already exists
    const nameQuery = query(studentsRef, orderByChild('name'), equalTo(name.value.trim()))
    const snapshot = await get(nameQuery)

    if (snapshot.exists()) {
      showPopupMessage(`Student name "${name.value}" already exists.`, 'error')
      loading.value = false
      return
    }

    // Prepare student data
    const sectionData = courseList.value[selectedCourse.value].sections[selectedSection.value]

    await push(studentsRef, {
      studentId: studentId.value,
      name: name.value.trim(),
      course: selectedCourse.value,
      section: selectedSection.value,
      startTime: sectionData.startTime,
      endTime: sectionData.endTime,
      timestamp: Date.now()
    })

    showPopupMessage('Student registered successfully!', 'success')

    // Reset form
    name.value = ''
    selectedCourse.value = ''
    selectedSection.value = ''
    studentId.value = generateRandomId()

  } catch (error) {
    console.error('Registration error:', error)
    showPopupMessage('Error registering student. Please try again.', 'error')
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
/* Popup animation */
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

.fixed {
  animation: fadeInUp 0.4s ease-out;
}
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

* {
  font-family: 'Poppins', sans-serif;
}
</style>
