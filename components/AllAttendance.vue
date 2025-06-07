<template>
  <transition name="fade" mode="out-in">
<main class="min-h-screen w-full flex flex-col font-poppins -mb-25">

     <img src="../assets/bg-2.png" alt="Form Header" class="w-full h-50 mb-5 rounded-xl" />
    <!-- Delete Confirmation Modal -->
<div v-if="showDeleteModal" class="fixed inset-0 bg-black bg-opacity-50 flex justify-center items-center z-50">
  <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-md text-center">
    <h2 class="text-xl font-bold mb-4">Delete Visible Records</h2>
    <p class="mb-6 text-gray-700">
      Are you sure you want to delete all visible attendance records? This action cannot be undone.
    </p>
    <div class="flex justify-center gap-4">
      <button
        @click="confirmDelete"
        class="bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700"
      >
        Yes, Delete
      </button>
      <button
        @click="showDeleteModal = false"
        class="bg-gray-300 text-gray-800 px-4 py-2 rounded hover:bg-gray-400"
      >
        Cancel
      </button>
    </div>
  </div>
</div>
   <div class="px-4 text-center sm:text-left">
  <h2
    class="text-2xl sm:text-3xl md:text-4xl font-bold mb-2 text-[#006699] break-words"
  >
    Welcome {{ teacherName }}!
  </h2>
  <p
    class="text-sm sm:text-base md:text-lg font-semibold mb-6 text-gray-700 leading-snug"
  >
    The following information displays a comprehensive list of student attendance records,
    including statuses and excuses, with real-time updating capabilities.
  </p>


    <!-- Filters -->
    <div class="mb-6 flex flex-wrap gap-4 items-center">
      <!-- Date Filters -->
      <label class="inline-flex items-center cursor-pointer select-none">
        <input type="radio" value="today" v-model="filterDate" class="sr-only peer" />
        <div
          class="w-5 h-5 mr-2 rounded-full border-2 border-gray-400
                 peer-checked:border-[#006699] peer-checked:bg-[#006699]
                 transition-colors"
        ></div>
        <span class="text-gray-700 peer-checked:text-[#006699]">Today</span>
      </label>

      <label class="inline-flex items-center cursor-pointer select-none">
        <input type="radio" value="week" v-model="filterDate" class="sr-only peer" />
        <div
          class="w-5 h-5 mr-2 rounded-full border-2 border-gray-400
                 peer-checked:border-[#006699] peer-checked:bg-[#006699]
                 transition-colors"
        ></div>
        <span class="text-gray-700 peer-checked:text-[#006699]">This Week</span>
      </label>

      <label class="inline-flex items-center cursor-pointer select-none">
        <input type="radio" value="all" v-model="filterDate" class="sr-only peer" />
        <div
          class="w-5 h-5 mr-2 rounded-full border-2 border-gray-400
                 peer-checked:border-[#006699] peer-checked:bg-[#006699]
                 transition-colors"
        ></div>
        <span class="text-gray-700 peer-checked:text-[#006699]">All Dates</span>
      </label>

      <!-- Section Filter -->
  <select v-model="filterSection" class="border rounded px-2 py-1 min-2xl">
    <option value="">All Sections</option>
    <option v-for="section in sectionsList" :key="section" :value="section">
      {{ section }}
    </option>
  </select>

    <!-- Export Button -->
 <button
  @click="exportToExcel"
  class="hidden md:inline-flex ml-auto bg-green-600 text-white px-4 py-2 rounded hover:bg-green-700 items-center gap-1"
>
  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none"
    viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
    <path stroke-linecap="round" stroke-linejoin="round"
      d="M4 16v2a2 2 0 002 2h12a2 2 0 002-2v-2M12 12v8m-4-4l4 4 4-4M12 4v8" />
  </svg>
  Export to Excel
</button>

<button
  @click="showDeleteModal = true"
  class="hidden md:inline-flex bg-red-600 text-white px-4 py-2 rounded hover:bg-red-700 items-center gap-1"
>
  <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none"
    viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
    <path stroke-linecap="round" stroke-linejoin="round"
      d="M6 18L18 6M6 6l12 12" />
  </svg>
  Delete All Visible
</button>

<!-- Mobile Icon Buttons -->
<button
  @click="exportToExcel"
  class="inline-flex md:hidden ml-auto bg-green-600 text-white p-2 rounded hover:bg-green-700"
  aria-label="Export to Excel"
>
  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none"
    viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
    <path stroke-linecap="round" stroke-linejoin="round"
      d="M4 16v2a2 2 0 002 2h12a2 2 0 002-2v-2M12 12v8m-4-4l4 4 4-4M12 4v8" />
  </svg>
</button>

<button
  @click="showDeleteModal = true"
  class="inline-flex md:hidden bg-red-600 text-white p-2 rounded hover:bg-red-700"
  aria-label="Delete All Visible"
>
  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none"
    viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
    <path stroke-linecap="round" stroke-linejoin="round"
      d="M6 18L18 6M6 6l12 12" />
  </svg>
</button>
    </div>

    <!-- Loading State -->
    <div v-if="loading" class="text-center">Loading attendance...</div>

    <!-- Attendance Table -->
    <div v-else class="overflow-x-auto">
      <table
        class="min-w-[1200px] w-full bg-white border border-gray-300 rounded-lg text-sm"
      >
        <thead>
          <tr class="bg-[#006699] text-white text-left">
            <th class="px-4 py-2">Date</th>
            <th class="px-4 py-2">Name</th>
            <th class="px-4 py-2">Course</th>
            <th class="px-4 py-2">Section</th>
            <th class="px-4 py-2">Start Time</th>
            <th class="px-4 py-2">End Time</th>
            <th class="px-4 py-2">Class Duration</th>
            <th class="px-4 py-2">Status</th>
            <th class="px-4 py-2">Excuse</th>
            <th class="px-4 py-2">Excuse Status</th>
            <th class="px-4 py-2">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(student, combinedKey) in filteredAttendance"
            :key="combinedKey"
            class="border-t hover:bg-gray-50"
          >
            <td class="px-4 py-2">{{ student.attendanceDate }}</td>
            <td class="px-4 py-2">{{ student.name }}</td>
            <td class="px-4 py-2">{{ student.course }}</td>
            <td class="px-4 py-2">{{ student.section }}</td>
            <td class="px-4 py-2">{{ student.startTime }}</td>
            <td class="px-4 py-2">{{ student.endTime }}</td>
            <td class="px-4 py-2">{{ student.timeAttended }}</td>
            <td class="px-4 py-2">
              <span
                class="px-2 py-1 rounded-full text-xs font-semibold"
                :class="{
                  'bg-green-100 text-green-700': student.status === 'present',
                  'bg-yellow-100 text-yellow-700': student.status === 'late',
                  'bg-red-100 text-red-700': student.status === 'absent',
                }"
              >
                {{ student.status }}
              </span>
            </td>
            <td class="px-4 py-2">
              <div v-if="student.excuse">
                <button
                  @click="openExcusePopup(student.excuseReason || 'No excuse reason provided')"
                  class="text-blue-600 hover:underline"
                >
                  View Excuse
                </button>
              </div>
              <div v-else>-</div>
            </td>
            <td class="px-4 py-2">
              <span
                class="px-2 py-1 rounded-full text-xs font-semibold capitalize"
                :class="{
                  'bg-yellow-100 text-yellow-700':
                    (student.excuseStatus || 'pending') === 'pending',
                  'bg-green-100 text-green-700':
                    (student.excuseStatus || 'pending') === 'approved',
                  'bg-red-100 text-red-700':
                    (student.excuseStatus || 'pending') === 'rejected',
                }"
              >
                {{ student.excuseStatus || 'pending' }}
              </span>
            </td>
            <td class="px-4 py-2">
              <select
                v-if="student.excuse"
                :value="student.excuseStatus || 'pending'"
                @change="onExcuseStatusChange(combinedKey, $event.target.value)"
                class="border rounded px-2 py-1 focus:outline-none focus:ring focus:border-blue-300"
              >
                <option value="pending">Pending</option>
                <option value="approved">Approve</option>
                <option value="rejected">Reject</option>
              </select>
              <span v-else>-</span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Excuse popup modal -->
    <div
      v-if="showPopup"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
    >
      <div class="bg-white rounded-lg p-6 max-w-lg w-full relative shadow-lg">
        <h3 class="text-xl font-semibold mb-4">Excuse Reason</h3>
        <p class="mb-6 whitespace-pre-wrap">{{ currentExcuseText }}</p>
        <button
          @click="closeExcusePopup"
          class="absolute top-2 right-2 text-gray-500 hover:text-gray-700 text-2xl font-extrabold"
          aria-label="Close"
        >
          &times;
        </button>
      </div>
    </div>
       </div>
  </main>
  
  </transition>
  
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, onValue, update } from 'firebase/database'
import { remove } from 'firebase/database'
const showDeleteModal = ref(false)
const isDeleting = ref(false)

const teacherName = ref('')

onMounted(() => {
  const storedName = localStorage.getItem('loggedInTeacher')
  if (storedName) {
    teacherName.value = storedName
  }
})

import {
  format,
  isWithinInterval,
  startOfWeek,
  endOfWeek,
  parse,
} from 'date-fns'
import * as XLSX from 'xlsx'

const firebaseConfig = {
  apiKey: 'AIzaSyB9VPCKfkTzEZR04BP5NZegF1cLfz4-dE4',
  authDomain: 'icct-attendancedb.firebaseapp.com',
  projectId: 'icct-attendancedb',
  storageBucket: 'icct-attendancedb.appspot.com',
  messagingSenderId: '575248656738',
  appId: '1:575248656738:web:097d086fca25a35e61e43b',
}

const app = getApps().length === 0 ? initializeApp(firebaseConfig) : getApps()[0]
const database = getDatabase(app)

const attendanceList = ref({})
const loading = ref(true)
const filterDate = ref('today')
const filterSection = ref('')
const showPopup = ref(false)
const currentExcuseText = ref('')

function parseTimeToDate(timeStr) {
  if (!timeStr) return null
  const now = new Date()
  try {
    return parse(timeStr, 'h:mm a', now)
  } catch {
    return null
  }
}
const sectionsList = computed(() => {
  const sections = new Set()
  for (const dateKey in attendanceList.value) {
    const studentsOnDate = attendanceList.value[dateKey]
    for (const studentKey in studentsOnDate) {
      const student = studentsOnDate[studentKey]
      if (student.section) sections.add(student.section)
    }
  }
  return Array.from(sections)
})


function calculateTimeAttended(startTime, endTime) {
  if (!startTime || !endTime) return '-'
  try {
    const start = parseTimeToDate(startTime)
    const end = parseTimeToDate(endTime)
    if (!start || !end) return '-'
    const diffMinutes = (end - start) / 60000
    if (diffMinutes < 0) return '-'
    const hours = Math.floor(diffMinutes / 60)
    const minutes = Math.floor(diffMinutes % 60)
    return `${hours}h ${minutes}m`
  } catch {
    return '-'
  }
}

function onExcuseStatusChange(combinedKey, newStatus) {
  // combinedKey is in format 'yyyy-MM-dd_studentUID'
  const splitIndex = combinedKey.indexOf('_')
  if (splitIndex === -1) {
    console.error('Invalid combinedKey format:', combinedKey)
    return
  }
  const dateKey = combinedKey.substring(0, splitIndex)
  const studentKey = combinedKey.substring(splitIndex + 1)

  // Reference to the student node for that date
  const studentRef = dbRef(database, `attendance/${dateKey}/${studentKey}`)

  update(studentRef, { excuseStatus: newStatus })
    .then(() => {
      // Update local data immediately
      if (
        attendanceList.value[dateKey] &&
        attendanceList.value[dateKey][studentKey]
      ) {
        attendanceList.value[dateKey][studentKey].excuseStatus = newStatus
      }
    })
    .catch((error) => {
      console.error('Failed to update excuse status:', error)
    })
}

function openExcusePopup(excuseText) {
  currentExcuseText.value = excuseText
  showPopup.value = true
}

function closeExcusePopup() {
  showPopup.value = false
  currentExcuseText.value = ''
}

// Load attendance data once on mounted
onMounted(() => {
  const attendanceRef = dbRef(database, 'attendance')
  onValue(attendanceRef, (snapshot) => {
    const data = snapshot.val() || {}
    attendanceList.value = data
    loading.value = false
  })
})

// Computed filtered attendance (flattened for easier v-for)
const filteredAttendance = computed(() => {
  const todayStr = format(new Date(), 'yyyy-MM-dd')
  const weekStart = startOfWeek(new Date(), { weekStartsOn: 1 }) // Monday
  const weekEnd = endOfWeek(new Date(), { weekStartsOn: 1 })

  let filtered = {}

  // Flatten attendanceList structure:
  // attendanceList: { date: { studentId: { ... } } }
  for (const dateKey in attendanceList.value) {
    // Filter by date
    if (filterDate.value === 'today' && dateKey !== todayStr) {
      continue
    } else if (filterDate.value === 'week') {
      const dateObj = new Date(dateKey)
      if (!(isWithinInterval(dateObj, { start: weekStart, end: weekEnd }))) {
        continue
      }
    }
    const studentsOnDate = attendanceList.value[dateKey]
    for (const studentKey in studentsOnDate) {
      const student = studentsOnDate[studentKey]

      // Filter by section if specified
      if (filterSection.value && student.section !== filterSection.value) {
        continue
      }

      // Calculate timeAttended and add attendanceDate
      const timeAttended = calculateTimeAttended(student.startTime, student.endTime)

      filtered[`${dateKey}_${studentKey}`] = {
        ...student,
        attendanceDate: dateKey,
        timeAttended,
      }
    }
  }

  return filtered
})


function exportToExcel() {
  // Prepare array of JSON objects for XLSX
  const exportData = Object.values(filteredAttendance.value).map((student) => ({
    Date: student.attendanceDate,
    Name: student.name,
    Course: student.course,
    Section: student.section,
    'Start Time': student.startTime,
    'End Time': student.endTime,
    'Time Attended': student.timeAttended,
    Status: student.status,
    Excuse: student.excuse ? 'Yes' : 'No',
    'Excuse Status': student.excuseStatus || 'pending',
  }))

  const worksheet = XLSX.utils.json_to_sheet(exportData)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, 'Attendance')

  XLSX.writeFile(workbook, 'attendance_records.xlsx')
}

function deleteVisibleAttendance() {
  const confirmation = confirm(
    'Are you sure you want to delete all visible attendance records? This cannot be undone.'
  )
  if (!confirmation) return

  const updates = []
  for (const combinedKey in filteredAttendance.value) {
    const splitIndex = combinedKey.indexOf('_')
    if (splitIndex === -1) continue

    const dateKey = combinedKey.substring(0, splitIndex)
    const studentKey = combinedKey.substring(splitIndex + 1)
    const studentRef = dbRef(database, `attendance/${dateKey}/${studentKey}`)

    updates.push(remove(studentRef))
  }

  Promise.all(updates)
    .then(() => {
      // Remove from local state manually
      for (const combinedKey in filteredAttendance.value) {
        const splitIndex = combinedKey.indexOf('_')
        if (splitIndex === -1) continue

        const dateKey = combinedKey.substring(0, splitIndex)
        const studentKey = combinedKey.substring(splitIndex + 1)

        if (attendanceList.value[dateKey]) {
          delete attendanceList.value[dateKey][studentKey]
          if (Object.keys(attendanceList.value[dateKey]).length === 0) {
            delete attendanceList.value[dateKey]
          }
        }
      }
    })
    .catch((error) => {
      console.error('Error deleting records:', error)
    })
}
async function confirmDelete() {
  isDeleting.value = true

  const updates = []
  for (const combinedKey in filteredAttendance.value) {
    const splitIndex = combinedKey.indexOf('_')
    if (splitIndex === -1) continue

    const dateKey = combinedKey.substring(0, splitIndex)
    const studentKey = combinedKey.substring(splitIndex + 1)
    const studentRef = dbRef(database, `attendance/${dateKey}/${studentKey}`)

    updates.push(remove(studentRef))
  }

  try {
    await Promise.all(updates)

    // Remove from local state
    for (const combinedKey in filteredAttendance.value) {
      const splitIndex = combinedKey.indexOf('_')
      if (splitIndex === -1) continue

      const dateKey = combinedKey.substring(0, splitIndex)
      const studentKey = combinedKey.substring(splitIndex + 1)

      if (attendanceList.value[dateKey]) {
        delete attendanceList.value[dateKey][studentKey]
        if (Object.keys(attendanceList.value[dateKey]).length === 0) {
          delete attendanceList.value[dateKey]
        }
      }
    }

    showDeleteModal.value = false
  } catch (error) {
    console.error('Error deleting records:', error)
    alert('An error occurred while deleting records.')
  } finally {
    isDeleting.value = false
  }
}

</script>

<style scoped>
* {
  font-family: 'Poppins', sans-serif;
 
}

</style>
