<template>
  <AppLayout>
    <div class="max-w-5xl mx-auto py-10 px-4">
      <header class="mb-10 text-center">
        <h1 class="text-3xl font-extrabold text-ink-gray-9">{{ __('Professional Network') }}</h1>
        <p class="text-ink-gray-5 mt-2">{{ __('Connect with partners, founders, and experts.') }}</p>
      </header>

      <div v-if="profiles.data" class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div v-for="profile in profiles.data" :key="profile.name" 
             class="p-6 bg-white border border-outline-gray-2 rounded-2xl shadow-sm hover:shadow-md transition-shadow">
          <Avatar :label="profile.user" size="xl" class="mb-4" />
          <h3 class="font-bold text-lg leading-tight">{{ profile.user }}</h3>
          <p class="text-blue-600 text-sm font-semibold mb-3">{{ profile.position }}</p>
          
          <div class="space-y-1 text-sm text-ink-gray-7 mb-6">
             <div class="flex items-center gap-2">📍 {{ profile.location }}</div>
             <div class="flex items-center gap-2">🏢 {{ profile.industry }}</div>
          </div>

          <div class="flex gap-2">
            <Button variant="solid" class="flex-1">{{ __('Connect') }}</Button>
            <Button v-if="profile.linkedin_url" :href="profile.linkedin_url" target="_blank">
               <Linkedin class="size-4" />
            </Button>
          </div>
        </div>
      </div>
    </div>
  </AppLayout>
</template>

<script setup>
import { createResource } from 'frappe-ui'
import { Linkedin } from 'lucide-vue-next'
import AppLayout from '@/components/Layouts/AppLayout.vue'

const profiles = createResource({
  url: 'data_platform.api.get_networking_profiles',
  auto: true,
})
</script>
