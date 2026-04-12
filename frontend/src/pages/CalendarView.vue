<template>
  <div class="p-8 max-w-6xl mx-auto">
    <div class="grid grid-cols-1 md:grid-cols-4 gap-4 mb-8">
      <div class="bg-white p-6 rounded-2xl border border-outline-gray-2 shadow-sm">
        <p class="text-xs font-bold text-ink-gray-4 uppercase tracking-wider">{{ __('Completed Total') }}</p>
        <p class="text-3xl font-bold text-ink-gray-9 mt-1">{{ metrics.data?.total_completed || 0 }}</p>
      </div>
      <div class="bg-white p-6 rounded-2xl border border-outline-gray-2 shadow-sm">
        <p class="text-xs font-bold text-ink-gray-4 uppercase tracking-wider">{{ __('Attended this Month') }}</p>
        <p class="text-3xl font-bold text-blue-600 mt-1">{{ metrics.data?.monthly_attended || 0 }}/8</p>
      </div>
      <div class="bg-white p-6 rounded-2xl border border-outline-gray-2 shadow-sm">
        <p class="text-xs font-bold text-ink-gray-4 uppercase tracking-wider">{{ __('Skips Used') }}</p>
        <p class="text-3xl font-bold mt-1" :class="metrics.data?.monthly_absent > 2 ? 'text-red-600' : 'text-ink-gray-9'">
          {{ metrics.data?.monthly_absent || 0 }}/2
        </p>
      </div>
      <div class="bg-blue-600 p-6 rounded-2xl shadow-md text-white">
        <p class="text-xs font-bold opacity-80 uppercase tracking-wider">{{ __('Status') }}</p>
        <p class="text-xl font-bold mt-1">
          {{ metrics.data?.monthly_absent >= 2 ? __('Limit Reached') : __('Active') }}
        </p>
      </div>
    </div>

    <div class="bg-white border border-outline-gray-2 rounded-3xl p-8 shadow-sm">
      <div class="flex items-center justify-between mb-8">
        <div>
          <h2 class="text-2xl font-bold text-ink-gray-9">{{ __("Lesson Schedule") }}</h2>
          <p class="text-sm text-ink-gray-4 mt-1">{{ currentMonthName }}</p>
        </div>
        <div class="flex gap-4 text-sm font-medium">
          <span class="flex items-center gap-2"><i class="w-3 h-3 bg-blue-600 rounded-full"></i> {{ __('Scheduled') }}</span>
          <span class="flex items-center gap-2"><i class="w-3 h-3 bg-green-500 rounded-full"></i> {{ __('Attended') }}</span>
          <span class="flex items-center gap-2"><i class="w-3 h-3 bg-red-500 rounded-full"></i> {{ __('Absent') }}</span>
        </div>
      </div>

      <div class="grid grid-cols-7 gap-2">
        <div v-for="day in ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun']" :key="day" class="text-center text-xs font-bold text-ink-gray-4 p-2">
          {{ day }}
        </div>

        <div v-for="blank in firstDayOfMonthOffset" :key="'blank-' + blank" class="h-24 bg-surface-gray-1/30 rounded-xl"></div>

        <div v-for="date in daysInMonth" 
             :key="date.format('YYYY-MM-DD')" 
             class="h-24 border border-outline-gray-1 rounded-xl p-2 relative transition-all hover:bg-surface-gray-1"
             :class="getLessonStatusClass(date)">
          
          <span class="text-xs font-bold" :class="isToday(date) ? 'bg-blue-600 text-white px-1.5 py-0.5 rounded-full' : ''">
            {{ date.date() }}
          </span>

          <div v-if="hasLesson(date)" class="mt-2 flex flex-col gap-1">
             <div v-for="lesson in getLessonsForDate(date)" :key="lesson.name" class="flex flex-col">
                <span class="text-[10px] font-black">{{ formatTime(lesson.lesson_date) }}</span>
                <span class="text-[9px] font-bold uppercase truncate leading-tight">{{ lesson.status }}</span>
             </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { createResource, dayjs } from 'frappe-ui'
import { computed } from 'vue'

// 1. Fetch Lessons
const lessons = createResource({
  url: 'lms.lms.api.get_student_lessons',
  auto: true,
})

// 2. Fetch Metrics
const metrics = createResource({
  url: 'lms.lms.api.get_attendance_metrics',
  auto: true,
})


const currentMonth = dayjs()
const currentMonthName = computed(() => currentMonth.format('MMMM YYYY'))

const firstDayOfMonthOffset = computed(() => {
  const firstDay = currentMonth.startOf('month').day() // 0 (Sun) to 6 (Sat)
  // Adjust so Monday is first (Frappe standard)
  return firstDay === 0 ? 6 : firstDay - 1
})

const daysInMonth = computed(() => {
  const count = currentMonth.daysInMonth()
  return Array.from({ length: count }, (_, i) => 
    currentMonth.startOf('month').add(i, 'day')
  )
})

const isToday = (date) => date.isSame(dayjs(), 'day')



const getLessonsForDate = (date) => {
  if (!lessons.data) return []
  return lessons.data.filter(l => {
    // We use lesson_date now as defined in the Python fields list
    if (!l.lesson_date) return false
    return dayjs(l.lesson_date).format('YYYY-MM-DD') === date.format('YYYY-MM-DD')
  })
}

const hasLesson = (date) => getLessonsForDate(date).length > 0

const getLessonStatusClass = (date) => {
  const dayLessons = getLessonsForDate(date)
  if (dayLessons.length === 0) return ''
  
  const status = dayLessons[0].status
  if (status === 'Attended') return 'bg-green-50 border-green-200 text-green-700'
  if (status === 'Absent') return 'bg-red-50 border-red-200 text-red-700'
  return 'bg-blue-50 border-blue-200 text-blue-700'
}

const formatTime = (datetime) => {
  return dayjs(datetime).format('HH:mm')
}
</script>
