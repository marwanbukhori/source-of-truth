<template>
  <div class="toc-container">
    <!-- Section Groups -->
    <div v-for="section in sections" :key="section.title" class="mb-8">
      <div class="section-header">
        <h2
          class="text-lg font-semibold text-gray-900 mb-4 cursor-pointer hover:text-gray-700"
          @click="$emit('category-click', section.title)"
        >
          {{ section.title }}
        </h2>

        <!-- Progress Indicator -->
        <div class="progress-bar">
          <div
            class="progress-fill bg-gray-600"
            :style="{ width: `${getProgressPercentage(section)}%` }"
          />
        </div>
      </div>

      <!-- Section Items -->
      <div class="space-y-2">
        <div
          v-for="item in section.items"
          :key="item.path"
          class="toc-item"
          :class="{ completed: isCompleted(item.path) }"
        >
          <!-- Item Content -->
          <div class="flex items-start p-3 rounded-lg hover:bg-gray-50">
            <!-- Status Icon -->
            <div class="mr-3 mt-1">
              <svg
                v-if="isCompleted(item.path)"
                class="w-5 h-5 text-green-500"
                fill="currentColor"
                viewBox="0 0 20 20"
              >
                <path
                  fill-rule="evenodd"
                  d="M10 18a8 8 0 100-16 8 8 0 000 16zm3.707-9.293a1 1 0 00-1.414-1.414L9 10.586 7.707 9.293a1 1 0 00-1.414 1.414l2 2a1 1 0 001.414 0l4-4z"
                  clip-rule="evenodd"
                />
              </svg>
              <div
                v-else
                class="w-5 h-5 border-2 border-gray-300 rounded-full"
              ></div>
            </div>

            <!-- Item Details -->
            <div class="flex-1">
              <div
                class="block cursor-pointer"
                @click="$emit('item-click', item.path)"
              >
                <h3 class="font-medium text-gray-900">{{ item.title }}</h3>
                <p class="text-sm text-gray-500">{{ item.description }}</p>
              </div>

              <!-- Tags/Metadata -->
              <div class="mt-2 flex flex-wrap gap-2">
                <span
                  v-for="tag in item.tags"
                  :key="tag"
                  class="inline-flex items-center px-2 py-1 rounded-full text-xs font-medium bg-gray-800 text-white"
                >
                  {{ tag }}
                </span>
              </div>
            </div>

            <!-- Actions -->
            <div class="ml-4 flex items-center space-x-2">
              <!-- Bookmark Button -->
              <button
                class="p-1 rounded-full hover:bg-gray-100"
                @click="$emit('toggle-bookmark', item)"
              >
                <svg
                  class="w-5 h-5"
                  :class="
                    item.isBookmarked
                      ? 'text-yellow-500 fill-current'
                      : 'text-gray-400'
                  "
                  viewBox="0 0 20 20"
                  fill="none"
                  stroke="currentColor"
                >
                  <path
                    d="M5 4a2 2 0 012-2h6a2 2 0 012 2v14l-5-2.5L5 18V4z"
                    :fill="item.isBookmarked ? 'currentColor' : 'none'"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                  />
                </svg>
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from "vue";
import { useRoute } from "vue-router";
import type {
  TableOfContentsItem,
  TableOfContentsSection,
} from "@/services/docs.service";

const props = defineProps<{
  sections: TableOfContentsSection[];
}>();

defineEmits<{
  (e: "item-click", path: string): void;
  (e: "category-click", category: string): void;
  (e: "toggle-bookmark", item: TableOfContentsItem): void;
}>();

const route = useRoute();

// Track completed items (could be stored in Vuex/Pinia)
const completedPaths = new Set<string>();

const isCompleted = (path: string): boolean => {
  return completedPaths.has(path);
};

const getProgressPercentage = (section: TableOfContentsSection): number => {
  const total = section.items.length;
  const completed = section.items.filter((item) =>
    isCompleted(item.path)
  ).length;
  return (completed / total) * 100;
};

// Active section based on current route
const activeSection = computed(() => {
  const currentPath = route.path;
  return props.sections.find((section) =>
    section.items.some((item) => item.path === currentPath)
  );
});

const isActiveItem = (item: TableOfContentsItem) => {
  return route.path === item.path;
};

const hasActiveItem = (section: TableOfContentsSection) => {
  return section.items.some((item) => isActiveItem(item));
};
</script>

<style scoped>
.toc-container {
  @apply px-4 py-6;
}

.section-header {
  @apply relative mb-4;
}

.progress-bar {
  @apply h-1 bg-gray-200 rounded-full overflow-hidden;
}

.progress-fill {
  @apply h-full transition-all duration-300;
}

.toc-item {
  @apply relative;
}

.toc-item.completed {
  @apply bg-gray-50;
}

.toc-item:not(:last-child)::after {
  content: "";
  @apply absolute left-5 top-8 bottom-0 w-0.5 bg-gray-200;
}
</style>
