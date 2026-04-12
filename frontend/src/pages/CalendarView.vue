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
        <h2 class="text-2xl font-bold text-ink-gray-9">{{ __("Lesson Schedule") }}</h2>
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
        <div v-for="n in 31" :key="n" 
             class="h-24 border border-outline-gray-1 rounded-xl p-2 relative transition-all hover:bg-surface-gray-1"
             :class="getLessonStatusClass(n)">
          <span class="text-xs font-bold">{{ n }}</span>
          <div v-if="hasLesson(n)" class="mt-2 text-[10px] font-bold uppercase truncate">
             {{ getLessonTitle(n) }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { createResource } from 'frappe-ui'
import { computed } from 'vue'

const lessons = createResource({
  url: 'lms.api.get_student_lessons',
  auto: true,
})

const metrics = createResource({
  url: 'lms.api.get_attendance_metrics',
  auto: true,
})

// Logic to style the calendar cells based on lesson status
const getLessonStatusClass = (day) => {
  const lesson = findLessonByDay(day)
  if (!lesson) return ''
  if (lesson.status === 'Attended') return 'bg-green-50 border-green-200 text-green-700'
  if (lesson.status === 'Absent') return 'bg-red-50 border-red-200 text-red-700'
  return 'bg-blue-50 border-blue-200 text-blue-700'
}

const findLessonByDay = (day) => {
  return lessons.data?.find(l => new Date(l.scheduled_date).getDate() === day)
}

const hasLesson = (day) => !!findLessonByDay(day)
const getLessonTitle = (day) => findLessonByDay(day)?.status
</script>
