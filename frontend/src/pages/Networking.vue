<template>
  <AppLayout :title="__('Professional Network')">
    <div class="max-w-6xl mx-auto py-12 px-6">
      <div class="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-6">
        <div>
          <h1 class="text-3xl font-extrabold text-ink-gray-9 tracking-tight">
            {{ __('Business Networking') }}
          </h1>
          <p class="text-ink-gray-5 mt-2 text-lg">
            {{ __('Find partners, co-founders, and experts in the Macunx community.') }}
          </p>
        </div>
        
        <div class="w-full md:w-80">
          <TextInput
            v-model="searchQuery"
            :placeholder="__('Search position or location...')"
            type="search"
          />
        </div>
      </div>

      <div v-if="profiles.loading" class="flex justify-center py-20">
        <LoadingIndicator class="w-10 h-10 text-blue-600" />
      </div>

      <div v-else-if="filteredProfiles.length" class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-8">
        <div 
          v-for="profile in filteredProfiles" 
          :key="profile.name"
          class="bg-white border border-outline-gray-2 rounded-3xl p-6 hover:shadow-xl hover:-translate-y-1 transition-all duration-300 group"
        >
          <div class="flex items-start justify-between mb-6">
            <Avatar 
              :label="profile.user" 
              size="2xl" 
              class="border-2 border-white shadow-sm" 
            />
            <a 
              v-if="profile.linkedin_url" 
              :href="profile.linkedin_url" 
              target="_blank"
              class="text-ink-gray-4 hover:text-blue-700 transition-colors"
            >
              <Linkedin class="w-6 h-6" />
            </a>
          </div>

          <div>
            <h3 class="text-xl font-bold text-ink-gray-9 mb-1">{{ profile.user }}</h3>
            <p class="text-blue-600 font-semibold text-sm uppercase tracking-wider">
              {{ profile.position }}
            </p>
            
            <div class="mt-4 space-y-2">
              <div class="flex items-center text-sm text-ink-gray-7">
                <span class="mr-2">🏢</span> {{ profile.industry }}
              </div>
              <div class="flex items-center text-sm text-ink-gray-7">
                <span class="mr-2">📍</span> {{ profile.location }}
              </div>
              <div class="mt-2 inline-block px-3 py-1 bg-gray-100 rounded-full text-xs font-medium text-gray-600">
                {{ profile.level }}
              </div>
            </div>
          </div>

          <div class="mt-8 pt-6 border-t border-gray-50">
            <Button variant="solid" class="w-full justify-center py-5 rounded-xl font-bold">
              {{ __('Request Connection') }}
            </Button>
          </div>
        </div>
      </div>

      <div v-else class="text-center py-20">
        <div class="text-5xl mb-4">🤝</div>
        <h3 class="text-xl font-bold">{{ __('No profiles found') }}</h3>
        <p class="text-gray-500">{{ __('Try adjusting your search or be the first to join!') }}</p>
      </div>
    </div>
  </AppLayout>
</template>

<script setup>
import AppLayout from '@/components/Layouts/AppLayout.vue'
import { ref, computed } from 'vue'
import { createResource, TextInput, Button, Avatar, LoadingIndicator } from 'frappe-ui'
import { Linkedin } from 'lucide-vue-next'


const searchQuery = ref('')

// Fetch profiles from your Data Platform API
const profiles = createResource({
  url: 'data_platform.api.get_networking_feed',
  auto: true,
})

// Simple search filter
const filteredProfiles = computed(() => {
  if (!profiles.data) return []
  return profiles.data.filter(p => {
    const search = searchQuery.value.toLowerCase()
    return (
      p.position?.toLowerCase().includes(search) ||
      p.location?.toLowerCase().includes(search) ||
      p.user?.toLowerCase().includes(search)
    )
  })
})
</script>
