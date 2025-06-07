<script setup>
import { ref, onMounted, computed, watch } from 'vue'
import { getApps, initializeApp } from 'firebase/app'
import { getDatabase, ref as dbRef, onValue, update, remove } from 'firebase/database'

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

const students = ref([])
const selectedCourse = ref('')
const selectedSection = ref('')

const courses = computed(() => {
  const courseSet = new Set()
  students.value.forEach(s => s.course && courseSet.add(s.course))
  return Array.from(courseSet).sort()
})

const sections = computed(() => {
  if (!selectedCourse.value) return []
  const sectionSet = new Set()
  students.value.forEach(s => s.course === selectedCourse.value && s.section && sectionSet.add(s.section))
  return Array.from(sectionSet).sort()
})

const filteredStudents = computed(() => {
  return students.value.filter(s => {
    const courseMatch = selectedCourse.value ? s.course === selectedCourse.value : true
    const sectionMatch = selectedSection.value ? s.section === selectedSection.value : true
    return courseMatch && sectionMatch
  })
})

watch(selectedCourse, (newCourse) => {
  if (!newCourse || !sections.value.includes(selectedSection.value)) {
    selectedSection.value = ''
  }
})

onMounted(() => {
  const studentsRef = dbRef(database, 'students')
  onValue(studentsRef, (snapshot) => {
    const data = snapshot.val()
    students.value = data ? Object.entries(data).map(([key, student]) => ({ id: key, ...student })) : []
  })
})

const showEditModal = ref(false)
const showDeleteModal = ref(false)
const showDeleteAllModal = ref(false)
const editStudent = ref({})
const studentToDelete = ref(null)

function openEditModal(student) {
  editStudent.value = { ...student }
  showEditModal.value = true
}

function submitEdit() {
  const studentRef = dbRef(database, `students/${editStudent.value.id}`)
  update(studentRef, {
    name: editStudent.value.name,
    course: editStudent.value.course,
    section: editStudent.value.section,
  }).then(() => {
    showEditModal.value = false
  })
}

function confirmDelete(student) {
  studentToDelete.value = student
  showDeleteModal.value = true
}

function deleteStudent() {
  const studentRef = dbRef(database, `students/${studentToDelete.value.id}`)
  remove(studentRef).then(() => {
    showDeleteModal.value = false
  })
}

function confirmDeleteAll() {
  showDeleteAllModal.value = true
}

function deleteAllStudents() {
  const studentsRef = dbRef(database, 'students')
  remove(studentsRef)
    .then(() => {
      showDeleteAllModal.value = false
    })
    .catch((error) => {
      console.error('Error deleting all students:', error)
    })
}
</script>

<template>
  <main class="min-h-screen items-center justify-center px-4 font-poppins -mb-30">
    <h1 class="text-4xl font-bold mb-2 text-[#006699]">STUDENT INFORMATION</h1>
    <p class="font-semibold mb-6">This section displays all the verified students with their alternative fingerprint data.</p>

    <!-- Filters + Button Section -->
    <div class="flex flex-col sm:flex-row gap-2 sm:gap-4 mb-6 items-stretch sm:items-center">
      <select v-model="selectedCourse" class="border border-gray-300 rounded px-3 py-2 w-full sm:w-auto">
        <option value="">All Courses</option>
        <option v-for="course in courses" :key="course" :value="course">{{ course }}</option>
      </select>

      <select v-model="selectedSection" :disabled="sections.length === 0" class="border border-gray-300 rounded px-3 py-2 w-full sm:w-auto">
        <option value="">All Sections</option>
        <option v-for="section in sections" :key="section" :value="section">{{ section }}</option>
      </select>

      <button
        @click="confirmDeleteAll"
        class="bg-red-600 hover:bg-red-700 text-white font-bold px-4 py-2 rounded w-full sm:w-auto flex justify-center items-center"
        :disabled="students.length === 0"
        title="Delete all students"
      >
        <!-- X for mobile -->
        <span class="inline-block sm:hidden text-white rounded px-2 h-8 select-none">Delete All Students</span>
        <!-- Full text for desktop -->
        <span class="hidden sm:inline h-8">Delete All Students</span>
      </button>
    </div>

    <!-- Table -->
    <div class="overflow-x-auto max-w-full">
      <table class="min-w-full table-auto text-sm text-left border border-gray-300 bg-white">
        <thead class="bg-[#006699] text-white">
          <tr>
            <th class="px-4 py-3 border-b">Student ID</th>
            <th class="px-4 py-3 border-b">Name</th>
            <th class="px-4 py-3 border-b">Course</th>
            <th class="px-4 py-3 border-b">Section</th>
            <th class="px-4 py-3 border-b">Start Time</th>
            <th class="px-4 py-3 border-b">End Time</th>
            <th class="px-4 py-3 border-b text-right">Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="filteredStudents.length === 0">
            <td colspan="7" class="px-4 py-3 text-center border">No students found.</td>
          </tr>
          <tr v-for="student in filteredStudents" :key="student.id" class="hover:bg-gray-100 even:bg-gray-50">
            <td class="px-4 py-3 border-b">{{ student.studentId }}</td>
            <td class="px-4 py-3 border-b">{{ student.name }}</td>
            <td class="px-4 py-3 border-b">{{ student.course }}</td>
            <td class="px-4 py-3 border-b">{{ student.section }}</td>
            <td class="px-4 py-3 border-b">{{ student.startTime }}</td>
            <td class="px-4 py-3 border-b">{{ student.endTime }}</td>
            <td class="px-4 py-3 border-b text-right space-x-2">
              <button @click="confirmDelete(student)" class="text-red-500 hover:text-red-700 font-bold">Delete</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Delete Single Modal -->
    <div v-if="showDeleteModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-sm shadow-lg animate-fade-in-up">
        <h2 class="text-lg font-semibold mb-4">Are you sure you want to delete?</h2>
        <div class="flex justify-end space-x-2">
          <button @click="showDeleteModal = false" class="bg-gray-300 px-4 py-2 rounded">Cancel</button>
          <button @click="deleteStudent" class="bg-red-500 text-white px-4 py-2 rounded">Delete</button>
        </div>
      </div>
    </div>

    <!-- Delete All Modal -->
    <div v-if="showDeleteAllModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white rounded-lg p-6 w-full max-w-sm shadow-lg animate-fade-in-up">
        <h2 class="text-lg font-semibold mb-4">Are you sure you want to delete <strong>all</strong> students?</h2>
        <div class="flex justify-end space-x-2">
          <button @click="showDeleteAllModal = false" class="bg-gray-300 px-4 py-2 rounded">Cancel</button>
          <button @click="deleteAllStudents" class="bg-red-600 text-white px-4 py-2 rounded">Delete All</button>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
* {
  font-family: 'Poppins', sans-serif;
}
</style>
