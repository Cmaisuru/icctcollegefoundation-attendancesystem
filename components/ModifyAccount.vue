<template>
  <main class="p-6 max-w-6xl mx-auto font-poppins relative">
    <img src="../assets/bg-4.png" alt="Form Header" class="w-full h-40" />
    <form @submit.prevent="submitAttendance" class="space-y-4 bg-white p-6 rounded shadow-md">
      <h2 class="text-4xl font-bold mb-3 text-[#006699] text-center">STUDENT PORTAL</h2>
      <p class="text-gray-700 text-center">
        Not feeling on time today? Submit a reasonable excuse to inform the instructors beforehand or right after the class.<br />
        Instruction: Please identify your information before submitting the document.
      </p>

      <!-- Date + Name -->
      <div class="flex flex-col md:flex-row gap-4">
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">Date</label>
          <input type="date" v-model="form.date" class="w-full border px-3 py-2 rounded" required />
        </div>
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">Name</label>
          <input v-model="form.name" type="text" class="w-full border px-3 py-2 rounded" required />
        </div>
      </div>

      <!-- Course + Section -->
      <div class="flex flex-col md:flex-row gap-4">
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">Course</label>
          <select v-model="form.course" @change="onCourseChange" class="w-full border px-3 py-2 rounded" required>
            <option disabled value="">Select Course</option>
            <option v-for="(course, key) in courses" :key="key" :value="key">{{ key }}</option>
          </select>
        </div>
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">Section</label>
          <select v-model="form.section" @change="onSectionChange" class="w-full border px-3 py-2 rounded" required>
            <option disabled value="">Select Section</option>
            <option v-for="(sectionData, sectionName) in sections" :key="sectionName" :value="sectionName">{{ sectionName }}</option>
          </select>
        </div>
      </div>

      <!-- Start Time + End Time -->
      <div class="flex flex-col md:flex-row gap-4">
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">Start Time</label>
          <input v-model="form.startTime" type="time" class="w-full border px-3 py-2 rounded bg-gray-100" readonly required />
        </div>
        <div class="flex-1 min-w-0">
          <label class="block font-semibold mb-1">End Time</label>
          <input v-model="form.endTime" type="time" class="w-full border px-3 py-2 rounded bg-gray-100" readonly required />
        </div>
      </div>

      <!-- Excuse Link -->
      <div>
        <label class="block font-semibold mb-1">Instructor Name</label>
        <input v-model="form.excuse" type="text" class="w-full border px-3 py-2 rounded" required/>
      </div>

      <!-- Excuse Reason -->
      <div>
        <label class="block font-semibold mb-1">Excuse Reason</label>
        <input v-model="form.excuseReason" class="w-full border px-3 py-2 rounded" required/>
      </div>

      <!-- Submit -->
      <button type="submit" class="bg-[#006699] text-white px-4 py-2 rounded hover:bg-[#005577]">
        Submit Attendance
      </button>
    </form>

    <!-- Popup Message -->
    <transition name="fade">
      <div
        v-if="popup.visible"
        :class="[
          'fixed bottom-8 left-1/2 transform -translate-x-1/2 px-6 py-3 rounded shadow-lg text-white font-semibold',
          popup.success ? 'bg-green-600' : 'bg-red-600'
        ]"
        style="z-index: 1000;"
      >
        {{ popup.message }}
      </div>
    </transition>
  </main>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, onValue, push, set } from 'firebase/database'
import { format } from 'date-fns'
import { get, child } from 'firebase/database'

// Firebase Init
const firebaseConfig = {
  apiKey: 'AIzaSyB9VPCKfkTzEZR04BP5NZegF1cLfz4-dE4',
  authDomain: 'icct-attendancedb.firebaseapp.com',
  projectId: 'icct-attendancedb',
  storageBucket: 'icct-attendancedb.appspot.com',
  messagingSenderId: '575248656738',
  appId: '1:575248656738:web:097d086fca25a35e61e43b',
}
const app = getApps().length ? getApps()[0] : initializeApp(firebaseConfig)
const database = getDatabase(app)

// Reactive Form State
const form = reactive({
  date: format(new Date(), 'yyyy-MM-dd'),
  name: '',
  course: '',
  section: '',
  startTime: '',
  endTime: '',
  excuse: '',
  excuseReason: '',
})

const courses = ref({})
const sections = ref({})

const popup = reactive({
  visible: false,
  message: '',
  success: true,
})

function showPopup(message, success = true) {
  popup.message = message
  popup.success = success
  popup.visible = true
  setTimeout(() => {
    popup.visible = false
  }, 3000)
}

const loadCourses = () => {
  const coursesRef = dbRef(database, 'courses')
  onValue(coursesRef, (snapshot) => {
    courses.value = snapshot.val() || {}
  })
}

const onCourseChange = () => {
  if (form.course && courses.value[form.course]) {
    sections.value = courses.value[form.course].sections || {}
    form.section = ''
    form.startTime = ''
    form.endTime = ''
  } else {
    sections.value = {}
    form.section = ''
    form.startTime = ''
    form.endTime = ''
  }
}

const onSectionChange = () => {
  const selected = courses.value[form.course]?.sections?.[form.section]
  if (selected) {
    form.startTime = convertTo24Hour(selected.startTime)
    form.endTime = convertTo24Hour(selected.endTime)
  } else {
    form.startTime = ''
    form.endTime = ''
  }
}

function convertTo24Hour(time12h) {
  if (!time12h) return ''
  const [time, modifier] = time12h.split(' ')
  let [hours, minutes] = time.split(':').map(Number)

  if (modifier === 'PM' && hours !== 12) hours += 12
  if (modifier === 'AM' && hours === 12) hours = 0

  return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}`
}

function formatTime(timeStr) {
  if (!timeStr) return ''
  const [hour, minute] = timeStr.split(':').map(Number)
  const date = new Date()
  date.setHours(hour, minute)
  return format(date, 'h:mm a')
}

const submitAttendance = async () => {
  const dateKey = form.date
  const attendanceRef = dbRef(database)
  const now = new Date()

  try {
    // 1. Fetch all attendance entries for the selected date
    const snapshot = await get(child(attendanceRef, `attendance/${dateKey}`))

    // 2. Check if the student's name already exists for that date
    let nameExists = false

    if (snapshot.exists()) {
      const attendanceEntries = snapshot.val()
      for (const key in attendanceEntries) {
        if (attendanceEntries[key].name?.toLowerCase() === form.name.trim().toLowerCase()) {
          nameExists = true
          break
        }
      }
    }

    if (nameExists) {
      showPopup('Attendance already submitted for this name today.', false)
      return
    }

    // 3. Calculate status
    const [startHour, startMinute] = form.startTime.split(':').map(Number)
    const startTimeDate = new Date(form.date)
    startTimeDate.setHours(startHour, startMinute, 0, 0)

    const diffMins = (now - startTimeDate) / 60000
    let status = 'present'

    if (form.excuse || form.excuseReason) {
      if (diffMins >= 120) status = 'absent'
      else if (diffMins >= 15) status = 'late'
    } else {
      if (diffMins >= 120) status = 'absent'
      else if (diffMins >= 15) status = 'late'
    }

    // 4. Prepare data
    const studentData = {
      name: form.name,
      course: form.course,
      section: form.section,
      startTime: formatTime(form.startTime),
      endTime: formatTime(form.endTime),
      status,
      timeAttended: format(now, 'h:mm a'),
      timestamp: now.toISOString(),
    }

    if (form.excuse || form.excuseReason) {
      studentData.excuse = form.excuse
      studentData.excuseReason = form.excuseReason
      studentData.excuseStatus = 'pending'
    }

    // 5. Submit new entry
    const newStudentRef = push(dbRef(database, `attendance/${dateKey}`))
    await set(newStudentRef, studentData)

    resetForm()
    showPopup('Student Form submitted successfully!', true)
  } catch (error) {
    console.error('Error:', error)
    showPopup('Failed to submit attendance. Please try again.', false)
  }
}
const resetForm = () => {
  form.date = format(new Date(), 'yyyy-MM-dd')
  form.name = ''
  form.course = ''
  form.section = ''
  form.startTime = ''
  form.endTime = ''
  form.excuse = ''
  form.excuseReason = ''
}

onMounted(() => {
  loadCourses()
})
</script>

<style scoped>
.font-poppins {
  font-family: 'Poppins', sans-serif;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.fade-enter-to,
.fade-leave-from {
  opacity: 1;
}
</style>
