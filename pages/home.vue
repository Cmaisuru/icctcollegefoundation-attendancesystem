<template>
  <transition name="fade" mode="out-in">
      <main class="font-poppins">
  <div class="min-h-screen flex flex-col bg-[#f0faff] text-[#003344] font-sans">

    <!-- Top header bar -->
    <header
      class="bg-[#006699] text-white flex items-center justify-between px-4 py-3 fixed top-0 left-0 right-0 z-50 md:justify-start"
    >
      <h1 class="text-lg font-semibold">ICCT CAINTA CAMPUS ATTENDANCE PORTAL</h1>
      <button
        @click="sidebarOpen = !sidebarOpen"
        aria-label="Toggle menu"
        class="focus:outline-none md:hidden"
      >
        <svg
          v-if="!sidebarOpen"
          class="w-6 h-6"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          viewBox="0 0 24 24"
        >
          <path d="M3 12h18M3 6h18M3 18h18" />
        </svg>
        <svg
          v-else
          class="w-6 h-6"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
          stroke-linecap="round"
          stroke-linejoin="round"
          viewBox="0 0 24 24"
        >
          <line x1="18" y1="6" x2="6" y2="18" />
          <line x1="6" y1="6" x2="18" y2="18" />
        </svg>
      </button>
    </header>

    <div class="flex flex-1 pt-14">
      <!-- Sidebar -->
      <nav
        :class="[ 'bg-white shadow-lg border-r border-gray-200 flex flex-col fixed top-14 left-0 h-[calc(100vh-3.5rem)] w-64 z-40 transform transition-transform duration-300 ease-in-out',
          sidebarOpen ? 'translate-x-0' : '-translate-x-full',
          'md:translate-x-0 md:static md:h-auto md:w-64'
        ]"
        @click.away="sidebarOpen = false"
      >
<div class="px-6 py-8 border-b border-gray-200 hidden md:block text-center">
  <!-- Logo image -->
  <img src="../assets/icct-colleges-rizal-philippines.jpg" alt="Logo" class="w-50 h-50 mx-auto" />
  <h2 class="text-xl font-semibold font-bold text-[#006699]">INSTRUCTOR TOOLS</h2>
</div>


        <ul class="flex flex-col space-y-2 p-6 text-gray-700">
          <li>
            <button
              @click="selectMenu('attendance')"
              :class="menuButtonClass('attendance')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              All Attendance
            </button>
          </li>
          <li>
            <button
              @click="selectMenu('create')"
              :class="menuButtonClass('create')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              Create Classes
            </button>
          </li>
          <li>
            <button
              @click="selectMenu('register')"
              :class="menuButtonClass('register')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              Register Student
            </button>
          </li>
          <li>
            <button
              @click="selectMenu('modify')"
              :class="menuButtonClass('modify')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              Student Portal
            </button>
          </li>
          <li>
            <button
              @click="selectMenu('view')"
              :class="menuButtonClass('view')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              View Classes
            </button>
          </li>
          <li>
            <button
              @click="selectMenu('allstudents')"
              :class="menuButtonClass('allstudents')"
              class="w-full font-semibold text-left px-4 py-3 rounded-lg transition-colors"
            >
              View All Students
            </button>
          </li>
        </ul>

        <div class="p-6 border-t border-gray-200">
          <button
            @click="confirmLogout = true; sidebarOpen = false"
            class="w-full bg-[#006699] text-white py-3 rounded-lg font-semibold hover:bg-[#004466] transition"
          >
            Logout
          </button>
        </div>
      </nav>

      <!-- Overlay for mobile sidebar -->
      <div
        v-if="sidebarOpen"
        @click="sidebarOpen = false"
        class="fixed inset-0 bg-black bg-opacity-40 z-30 md:hidden"
      ></div>

      <!-- Main Content Area -->
      <main class="flex-1 p-6 overflow-auto">
        <AllAttendance
          v-if="selectedMenu === 'attendance'"
          :students="students"
          :teacherName="teacherName"
        />
        <CreateClass
          v-else-if="selectedMenu === 'create'"
          :teacherName="teacherName"
        />
        <RegisterStudent
          v-else-if="selectedMenu === 'register'"
          :teacherName="teacherName"
        />
        <ModifyAccount
          v-else-if="selectedMenu === 'modify'"
          :teacherName="teacherName"
        />
        <ViewData
          v-else-if="selectedMenu === 'view'"
          :students="students"
          :teacherName="teacherName"
          :selectedSection="selectedSection"
          @update:selectedSection="selectedSection = $event"
        />
        <AllStudents
          v-else-if="selectedMenu === 'allstudents'"
          :students="students"
          :teacherName="teacherName"
        />
      </main>
    </div>

    <!-- Logout Confirmation Modal -->
    <div
      v-if="confirmLogout"
      class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50"
    >
      <div class="bg-white rounded-xl p-8 max-w-sm w-full shadow-lg text-center">
        <h3 class="text-lg font-bold mb-4">Confirm Logout</h3>
        <p class="mb-6">Are you sure you want to logout?</p>
        <div class="flex justify-center gap-6">
          <button
            @click="logout"
            class="bg-[#006699] text-white px-5 py-2 rounded-lg hover:bg-[#004466] transition"
          >
            Yes
          </button>
          <button
            @click="confirmLogout = false"
            class="bg-gray-300 text-gray-700 px-5 py-2 rounded-lg hover:bg-gray-400 transition"
          >
            No
          </button>
        </div>
      </div>
    </div>
  </div>
      </main>
      </transition>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'

import AllAttendance from '../components/AllAttendance.vue'
import CreateClass from '../components/CreateClass.vue'
import RegisterStudent from '../components/RegisterStudent.vue'
import ModifyAccount from '../components/ModifyAccount.vue'
import ViewData from '../components/ViewData.vue'
import AllStudents from '../components/AllStudents.vue'

const router = useRouter()
const teacherName = ref('')
const confirmLogout = ref(false)
const selectedSection = ref('')
const selectedMenu = ref('attendance') // Default menu
const sidebarOpen = ref(false)

onMounted(() => {
  const storedName = localStorage.getItem('teacherName')
  teacherName.value = storedName || 'Teacher'
})

function logout() {
  localStorage.clear()
  router.push('/')
}

function selectMenu(menu) {
  selectedMenu.value = menu
  sidebarOpen.value = false
}

function menuButtonClass(menu) {
  return selectedMenu.value === menu
    ? 'bg-[#006699] text-white font-semibold'
    : 'hover:bg-[#cce6ff]'
}
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
}
.fade-enter-to, .fade-leave-from {
  opacity: 1;
}

.pt-14 {
  padding-top: 3.5rem; /* Matches header height */
}

</style>
