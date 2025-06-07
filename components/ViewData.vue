<template>
<main class="min-h-screen items-center justify-center px-4 font-poppins -mb-30">
  <!-- Validation Message Popup -->
<div
  v-if="validationMessage"
  class="fixed top-4 left-1/2 transform -translate-x-1/2 bg-red-100 text-red-800 border border-red-300 px-4 py-2 rounded shadow z-50"
>
  {{ validationMessage }}
</div>

    <h1 class="text-4xl font-bold mb-2 text-[#006699]">ALL CLASSES</h1>
      <p class="font-semibold mb-6">This section displays all the registered courses and sections, couldn't find the course you need? Create a New Class under "Create New Classes".</p>
    <div>
      <!-- Filter by Course -->
      <div class="mb-4 max-w-sm">
        <label class="block mb-1 font-semibold">Filter by Course</label>
        <select v-model="selectedCourseFilter" class="w-full border border-gray-300 rounded px-3 py-2">
          <option value="">All Courses</option>
          <option v-for="(data, courseName) in courseList" :key="courseName" :value="courseName">
            {{ courseName }}
          </option>
        </select>
      </div>

      <div v-if="Object.keys(courseList).length === 0" class="text-center text-gray-500">
        No courses available.. Please add more courses in "Create Classes"
      </div>

      <div v-else class="overflow-x-auto">
        <table class="min-w-full bg-white border border-gray-300 rounded-lg text-sm sm:text-base">
          <thead>
            <tr class="bg-[#006699] text-white text-left">
              <th class="px-4 py-2">Course</th>
              <th class="px-4 py-2">Section</th>
              <th class="px-4 py-2">Start Time</th>
              <th class="px-4 py-2">End Time</th>
              <th class="px-4 py-2 text-right">Actions</th>
            </tr>
          </thead>
          <tbody>
            <template v-for="(courseData, courseName) in filteredCourses" :key="courseName">
             <tr class="bg-[#006699]/10 text-[#006699] font-semibold">
  <td colspan="4" class="px-4 py-2">{{ courseName }}</td>
  <td class="px-4 py-2 text-right">
    <button
      @click="handleDeleteCourse(courseName)"
      class="text-red-600 hover:underline text-sm sm:text-base"
    >
       Delete Entire Course
    </button>
                </td>
              </tr>
              <tr
                v-for="(sectionData, sectionName) in courseData.sections"
                :key="sectionName"
                class="border-t hover:bg-gray-50"
              >
                <td class="px-4 py-2">{{ courseName }}</td>
                <td class="px-4 py-2">{{ sectionName }}</td>
                <td class="px-4 py-2">{{ sectionData.startTime }}</td>
                <td class="px-4 py-2">{{ sectionData.endTime }}</td>
                <td class="px-4 py-2 text-right space-x-2">
                  <button @click="openEditModal(courseName, sectionName, sectionData)" class="text-blue-600 hover:underline">
                    ✏️
                  </button>
                  <button @click="handleDeleteSection(courseName, sectionName)" class="text-red-600 hover:underline">
                    🗑️
                  </button>
                </td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>

      <!-- 🧩 Edit Modal -->
      <transition name="fade-scale">
        <div v-if="showModal" class="fixed inset-0 bg-opacity-50 flex items-center justify-center z-50">
          <div class="bg-white p-6 rounded-lg w-full max-w-md shadow-lg transform scale-95 animate-pop">
            <h3 class="text-xl font-bold mb-4 text-[#006699]">Edit Class</h3>
            <p class="text-gray-700 mb-4">Editing <strong>{{ currentCourse }}</strong> - <strong>{{ currentSection }}</strong></p>

            <div class="mb-4">
              <label class="block mb-1 font-semibold">Start Time</label>
              <input v-model="editStartTime" type="time" class="w-full border border-gray-300 rounded px-3 py-2" required/>
            </div>

            <div class="mb-4">
              <label class="block mb-1 font-semibold">End Time</label>
              <input v-model="editEndTime" type="time" required class="w-full border border-gray-300 rounded px-3 py-2"/>
            </div>

            <div class="flex justify-end space-x-2">
              <button @click="closeModal" class="px-4 py-2 rounded bg-gray-300 hover:bg-gray-400 text-gray-800">
                Cancel
              </button>
              <button @click="saveChanges" class="px-4 py-2 rounded bg-[#006699] hover:bg-[#004466] text-white">
                Save
              </button>
            </div>
          </div>
        </div>
      </transition>

      <!-- ❗ Delete Confirmation Modal -->
      <transition name="fade-scale">
        <div v-if="showDeleteModal" class="fixed inset-0 bg-opacity-50 flex items-center justify-center z-50">
          <div class="bg-white p-6 rounded-lg max-w-md shadow-lg transform scale-95 animate-pop">
            <h3 class="text-xl font-bold mb-4 text-red-600">Confirm Deletion</h3>
            <p class="text-gray-700 mb-4">
              Are you sure you want to delete <strong>{{ sectionToDelete }}</strong> from <strong>{{ courseToDelete }}</strong>?
            </p>
            <div class="flex justify-end space-x-2">
              <button @click="showDeleteModal = false" class="px-4 py-2 rounded bg-gray-300 hover:bg-gray-400 text-gray-800">
                Cancel
              </button>
              <button @click="confirmDeleteSection" class="px-4 py-2 rounded bg-red-600 hover:bg-red-700 text-white">
                Yes, Delete
              </button>
            </div>
          </div>
        </div>
      </transition>

      <!-- ❗ Delete Course Confirmation Modal -->
      <transition name="fade-scale">
        <div v-if="showDeleteCourseModal" class="fixed inset-0 bg-opacity-50 flex items-center justify-center z-50">
          <div class="bg-white p-6 rounded-lg max-w-md shadow-lg transform scale-95 animate-pop">
            <h3 class="text-xl font-bold mb-4 text-red-600">Confirm Course Deletion</h3>
            <p class="text-gray-700 mb-4">
              Are you sure you want to delete the entire course <strong>{{ courseToDelete }}</strong>?
            </p>
            <div class="flex justify-end space-x-2">
              <button @click="showDeleteCourseModal = false" class="px-4 py-2 rounded bg-gray-300 hover:bg-gray-400 text-gray-800">
                Cancel
              </button>
              <button @click="confirmDeleteCourse" class="px-4 py-2 rounded bg-red-600 hover:bg-red-700 text-white">
                Yes, Delete
              </button>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </main>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, onValue, remove, update } from 'firebase/database'
const validationMessage = ref('')

// Firebase config
const firebaseConfig = {
  apiKey: "AIzaSyB9VPCKfkTzEZR04BP5NZegF1cLfz4-dE4",
  authDomain: "icct-attendancedb.firebaseapp.com",
  projectId: "icct-attendancedb",
  storageBucket: "icct-attendancedb.appspot.com",
  messagingSenderId: "575248656738",
  appId: "1:575248656738:web:097d086fca25a35e61e43b"
}

// Init Firebase
const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
const database = getDatabase(app)

// Data and states
const courseList = ref({})
const selectedCourseFilter = ref('')
const showModal = ref(false)
const currentCourse = ref('')
const currentSection = ref('')
const editStartTime = ref('')
const editEndTime = ref('')
const showDeleteModal = ref(false)
const showDeleteCourseModal = ref(false)
const courseToDelete = ref('')
const sectionToDelete = ref('')

// On mount - fetch data
onMounted(() => {
  const courseRef = dbRef(database, 'courses')
  onValue(courseRef, snapshot => {
    courseList.value = snapshot.exists() ? snapshot.val() : {}
  })
})

// Filtered list
const filteredCourses = computed(() => {
  if (!selectedCourseFilter.value) return courseList.value
  const course = courseList.value[selectedCourseFilter.value]
  return course && course.sections
    ? { [selectedCourseFilter.value]: course }
    : {}
})

// Open edit
function openEditModal(courseName, sectionName, sectionData) {
  currentCourse.value = courseName
  currentSection.value = sectionName
  editStartTime.value = sectionData.startTime || ''
  editEndTime.value = sectionData.endTime || ''
  showModal.value = true
}

// Close edit modal
function closeModal() {
  showModal.value = false
}

function formatTimeToAMPM(time24) {
  const [hourStr, minute] = time24.split(':')
  let hour = parseInt(hourStr)
  const ampm = hour >= 12 ? 'PM' : 'AM'
  hour = hour % 12 || 12
  return `${hour.toString().padStart(2, '0')}:${minute} ${ampm}`
}

function showValidation(msg) {
  validationMessage.value = msg
  setTimeout(() => {
    validationMessage.value = ''
  }, 3000) // Hide after 3 seconds
}

function saveChanges() {
  const start = editStartTime.value
  const end = editEndTime.value

  if (!start || !/^\d{2}:\d{2}$/.test(start)) {
    showValidation("Please set a valid Start Time before saving.")
    return
  }

  if (!end || !/^\d{2}:\d{2}$/.test(end)) {
    showValidation("Please set a valid End Time before saving.")
    return
  }

  const sectionPath = `courses/${currentCourse.value}/sections/${currentSection.value}`

  const formattedStart = formatTimeToAMPM(start)
  const formattedEnd = formatTimeToAMPM(end)

  update(dbRef(database, sectionPath), {
    startTime: formattedStart,
    endTime: formattedEnd
  })
    .then(() => {
      closeModal()
    })
}

// Ask for section delete confirmation
function handleDeleteSection(courseName, sectionName) {
  courseToDelete.value = courseName
  sectionToDelete.value = sectionName
  showDeleteModal.value = true
}

// Confirm section delete with auto course delete if last section
function confirmDeleteSection() {
  const sectionRef = dbRef(database, `courses/${courseToDelete.value}/sections/${sectionToDelete.value}`)
  const courseRef = dbRef(database, `courses/${courseToDelete.value}`)

  // Delete the section first
  remove(sectionRef)
    .then(() => {
      const remainingSectionsRef = dbRef(database, `courses/${courseToDelete.value}/sections`)
      onValue(remainingSectionsRef, snapshot => {
        if (!snapshot.exists()) {
          // If no remaining sections, delete the whole course
          remove(courseRef)
            .then(() => {
              showDeleteModal.value = false
            })
        } else {
          showDeleteModal.value = false
        }
      }, { onlyOnce: true })
    })
}

// Ask for entire course delete confirmation
function handleDeleteCourse(courseName) {
  courseToDelete.value = courseName
  showDeleteCourseModal.value = true
}

// Confirm entire course delete
function confirmDeleteCourse() {
  const courseRef = dbRef(database, `courses/${courseToDelete.value}`)
  remove(courseRef)
    .then(() => {
      showDeleteCourseModal.value = false
    })
}
</script>

<style scoped>
/* Modal transition */
.fade-scale-enter-active,
.fade-scale-leave-active {
  transition: all 0.3s ease;
}
.fade-scale-enter-from,
.fade-scale-leave-to {
  opacity: 0;
  transform: scale(0.9);
}
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');

.font-poppins {
  font-family: 'Poppins', sans-serif;
}

/* Pop animation */
@keyframes pop {
  0% {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
.animate-pop {
  animation: pop 0.3s ease-out;
}
</style>
