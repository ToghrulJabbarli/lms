<template>
  <div class="p-8 max-w-6xl mx-auto">
    <header class="mb-10 flex flex-col md:flex-row md:items-end justify-between gap-4">
      <div>
        <h1 class="text-3xl font-bold text-ink-gray-9 tracking-tight">{{ __('Networking') }}</h1>
        <p class="text-ink-gray-5 mt-2">{{ __('Connect and collaborate with Macunx professionals.') }}</p>
      </div>
      
      <div class="w-full max-w-sm relative">
        <TextInput
          v-model="searchQuery"
          :placeholder="__('Search name, role, or industry...')"
          type="search"
          size="sm"
        >
          <template #prefix>
            <Search class="h-4 w-4 text-ink-gray-4" />
          </template>
        </TextInput>
      </div>
    </header>

    <div v-if="searchQuery" class="mb-6 flex items-center justify-between bg-surface-gray-2 p-3 rounded-lg border border-outline-gray-1">
      <p class="text-sm text-ink-gray-7">
        {{ __('Showing results for') }} 
        <span class="font-bold text-ink-gray-9 italic">"{{ searchQuery }}"</span>
      </p>
      <button 
        @click="searchQuery = ''" 
        class="text-xs text-blue-600 hover:underline font-medium"
      >
        {{ __('Clear Search') }}
      </button>
    </div>

    <div v-if="profiles.loading" class="flex flex-col items-center justify-center py-32 space-y-4">
      <LoadingIndicator class="w-10 h-10 text-blue-600" />
      <p class="text-ink-gray-4 animate-pulse">{{ __('Fetching community profiles...') }}</p>
    </div>

    <div v-else-if="filteredProfiles.length" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
      <div 
        v-for="profile in filteredProfiles" 
        :key="profile.name"
        class="group bg-white border border-outline-gray-2 rounded-2xl p-6 shadow-sm hover:shadow-xl hover:border-blue-200 transition-all duration-300"
      >
        <div class="flex items-start gap-4 mb-4">
          <Avatar 
            :label="profile.user" 
            size="2xl" 
            class="group-hover:scale-105 transition-transform"
          />
          <div class="overflow-hidden">
            <h3 class="font-bold text-ink-gray-9 truncate" :title="profile.user">{{ profile.user }}</h3>
            <p class="text-sm text-blue-600 font-semibold truncate">{{ profile.position || __('Member') }}</p>
            <div class="mt-1 flex items-center text-xs text-ink-gray-4">
               <span class="bg-surface-gray-2 px-2 py-0.5 rounded-full">{{ profile.industry }}</span>
            </div>
          </div>
        </div>
        
        <div class="grid grid-cols-2 gap-2 text-sm text-ink-gray-7 mb-6 py-4 border-y border-outline-gray-1 border-dashed">
          <div class="flex items-center gap-2">
            <span class="text-lg">📍</span> 
            <span class="truncate">{{ profile.location || __('Remote') }}</span>
          </div>
          <div class="flex items-center gap-2">
            <span class="text-lg">💼</span> 
            <span class="truncate">{{ profile.industry }}</span>
          </div>
        </div>

        <Button variant="solid" class="w-full justify-center shadow-sm">
          {{ __('View Full Profile') }}
        </Button>
      </div>
    </div>

    <div v-else class="text-center py-24 bg-surface-gray-1 rounded-3xl border-2 border-dashed border-outline-gray-2">
      <div class="inline-flex items-center justify-center w-16 h-16 bg-white rounded-full shadow-sm mb-4">
        <Users v-if="!searchQuery" class="h-8 w-8 text-ink-gray-3" />
        <Search v-else class="h-8 w-8 text-ink-gray-3" />
      </div>
      <h2 class="text-xl font-semibold text-ink-gray-9">
        {{ searchQuery ? __('No matches found') : __('No profiles yet') }}
      </h2>
      <p class="text-ink-gray-5 mt-2 max-w-xs mx-auto">
        {{ searchQuery 
          ? __('Try adjusting your keywords or filters to find what you are looking for.') 
          : __('Be the first to join the network! Create your profile in the dashboard.') 
        }}
      </p>
      <Button 
        v-if="searchQuery" 
        variant="ghost" 
        class="mt-6" 
        @click="searchQuery = ''"
      >
        {{ __('Back to all profiles') }}
      </Button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import { createResource, TextInput, Button, Avatar, LoadingIndicator } from 'frappe-ui'
import { Search, Users } from 'lucide-vue-next' // Ensure these icons are available

const searchQuery = ref('')

const profiles = createResource({
  url: 'lms.api.get_networking_feed', 
  auto: true,
})

const filteredProfiles = computed(() => {
  const allProfiles = profiles.data || []
  const q = searchQuery.value.toLowerCase().trim()
  
  if (!q) return allProfiles
  
  return allProfiles.filter(p => {
    return [p.user, p.position, p.industry, p.location].some(field => 
      (field || '').toLowerCase().includes(q)
    )
  })
})
</script>
