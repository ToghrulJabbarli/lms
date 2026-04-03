<template>
  <div class="p-8 max-w-5xl mx-auto">
    <header class="mb-10">
      <h1 class="text-3xl font-bold text-ink-gray-9 tracking-tight">{{ __('Networking') }}</h1>
      <p class="text-ink-gray-5 mt-2">{{ __('Build business partnerships within the Macunx community.') }}</p>
    </header>

    <div class="mb-8 w-full max-w-sm">
      <TextInput
        v-model="searchQuery"
        :placeholder="__('Search by position or industry...')"
        type="search"
      />
    </div>

    <div v-if="profiles.loading" class="flex justify-center py-20">
      <LoadingIndicator class="w-8 h-8 text-blue-600" />
    </div>

    <div v-else-if="filteredProfiles.length" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div 
        v-for="profile in filteredProfiles" 
        :key="profile.name"
        class="bg-white border border-outline-gray-2 rounded-2xl p-6 shadow-sm hover:shadow-md transition-all"
      >
        <div class="flex items-center gap-4 mb-4">
          <Avatar :label="profile.user" size="xl" />
          <div>
            <h3 class="font-bold text-ink-gray-9">{{ profile.user }}</h3>
            <p class="text-sm text-blue-600 font-semibold">{{ profile.position }}</p>
          </div>
        </div>
        
        <div class="space-y-2 text-sm text-ink-gray-7 mb-6">
          <div class="flex items-center gap-2">📍 {{ profile.location || __('Remote') }}</div>
          <div class="flex items-center gap-2">🏢 {{ profile.industry }}</div>
        </div>

        <Button variant="solid" class="w-full justify-center">
          {{ __('View Profile') }}
        </Button>
      </div>
    </div>

    <div v-else class="text-center py-20 bg-surface-gray-1 rounded-2xl">
      <p class="text-ink-gray-4">{{ __('No partners found matching your search.') }}</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { createResource, TextInput, Button, Avatar, LoadingIndicator } from 'frappe-ui'

// We removed the Layout imports entirely because App.vue handles them globally.
const searchQuery = ref('')

const profiles = createResource({
  url: 'data_platform.api.get_networking_feed',
  auto: true,
})

const filteredProfiles = computed(() => {
  if (!profiles.data) return []
  const q = searchQuery.value.toLowerCase()
  return profiles.data.filter(p => 
    (p.user || '').toLowerCase().includes(q) || 
    (p.position || '').toLowerCase().includes(q)
  )
})
</script>
