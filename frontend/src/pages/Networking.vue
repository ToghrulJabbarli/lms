<template>
  <AppLayout>
    <div class="max-w-5xl mx-auto py-10 px-6">
      <div class="mb-8">
        <h1 class="text-3xl font-bold text-gray-900">{{ __('Professional Network') }}</h1>
        <p class="text-gray-600">{{ __('Connect with the Macunx community') }}</p>
      </div>

      <div class="mb-10 w-full max-w-sm">
        <TextInput
          v-model="searchQuery"
          :placeholder="__('Search by position...')"
        />
      </div>

      <div v-if="profiles.loading" class="text-center py-10">
        <LoadingIndicator class="w-8 h-8 mx-auto" />
      </div>

      <div v-else-if="filteredProfiles.length" class="grid grid-cols-1 md:grid-cols-3 gap-6">
        <div 
          v-for="profile in filteredProfiles" 
          :key="profile.name"
          class="p-6 bg-white border border-gray-200 rounded-xl shadow-sm"
        >
          <div class="flex items-center space-x-4 mb-4">
            <Avatar :label="profile.user" size="xl" />
            <div>
              <h3 class="font-bold text-gray-900">{{ profile.user }}</h3>
              <p class="text-sm text-blue-600 font-medium">{{ profile.position }}</p>
            </div>
          </div>
          
          <div class="text-sm text-gray-600 space-y-1 mb-6">
            <p>🏢 {{ profile.industry }}</p>
            <p>📍 {{ profile.location }}</p>
          </div>

          <Button variant="solid" class="w-full">
            {{ __('View Profile') }}
          </Button>
        </div>
      </div>

      <div v-else class="text-center py-20 bg-gray-50 rounded-2xl">
        <p class="text-gray-500">{{ __('No profiles found matching your search.') }}</p>
      </div>
    </div>
  </AppLayout>
</template>

<script setup>
import { ref, computed } from 'vue'
import { createResource, TextInput, Button, Avatar, LoadingIndicator } from 'frappe-ui'
// Icons removed to prevent build errors if lucide version is mismatched
import AppLayout from '../components/Layouts/AppLayout.vue' 

const searchQuery = ref('')

const profiles = createResource({
  url: 'data_platform.api.get_networking_feed',
  auto: true,
})

const filteredProfiles = computed(() => {
  if (!profiles.data) return []
  const q = searchQuery.value.toLowerCase()
  return profiles.data.filter(p => 
    (p.position || '').toLowerCase().includes(q) || 
    (p.user || '').toLowerCase().includes(q)
  )
})
</script>
