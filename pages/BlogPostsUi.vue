<template>
  <div class="min-h-screen bg-black p-10 text-white">
    <div class="max-w-6xl mx-auto mt-20 space-y-10">
      <div class="text-center">
        <h1 class="text-3xl font-semibold">Список постів блогу</h1>
      </div>

      <div class="flex justify-between items-center">
        <div class="text-gray-400 text-sm">
          Всього постів: {{ totalPosts }}
        </div>
      </div>

      <div class="w-full space-y-4 pb-4">
        <UTable
            :data="posts"
            :columns="columns"
            :loading="loading"
            class="flex-1 bg-gray-900 border border-gray-700 rounded-lg"
            @row-click="goToBlog"
        />

        <div class="flex justify-center gap-2 pt-4" v-if="totalPages > 1">
          <button
              @click="goToPage(1)"
              :disabled="currentPage === 1"
              class="px-3 py-2 bg-blue-600 text-white rounded disabled:bg-gray-600"
          >
            Перша
          </button>

          <button
              @click="goToPage(currentPage - 1)"
              :disabled="currentPage === 1"
              class="px-3 py-2 bg-blue-600 text-white rounded disabled:bg-gray-600"
          >
            Попередня
          </button>

          <span class="px-3 py-2 bg-gray-700 text-white rounded">
            {{ currentPage }} / {{ totalPages }}
          </span>

          <button
              @click="goToPage(currentPage + 1)"
              :disabled="currentPage === totalPages"
              class="px-3 py-2 bg-blue-600 text-white rounded disabled:bg-gray-600"
          >
            Наступна
          </button>

          <button
              @click="goToPage(totalPages)"
              :disabled="currentPage === totalPages"
              class="px-3 py-2 bg-blue-600 text-white rounded disabled:bg-gray-600"
          >
            Остання
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'
import { navigateTo } from '#app'
import { h } from 'vue'
import { NuxtLink } from '#components'

const config = useRuntimeConfig()

type Post = {
  id: number
  title: string
  user: { name: string }
  category: { title: string }
  published_at: string
}

interface ApiResponse {
  data: Post[]
  meta: {
    current_page: number
    last_page: number
    total: number
    per_page: number
  }
}

const posts = ref<Post[]>([])
const loading = ref(false)
const currentPage = ref(1)
const totalPosts = ref(0)
const totalPages = ref(1)
const pageSize = 10

const goToBlog = (row: Post) => {
  if (row && row.id) {
    navigateTo(`/blog/${row.id}`)
  }
}

const goToPage = async (page: number) => {
  if (page < 1 || page > totalPages.value || page === currentPage.value) return

  currentPage.value = page
  await getPosts(page)
}

const getPosts = async (page: number = 1) => {
  loading.value = true

  try {
    const response = await $fetch<ApiResponse>(
        `${config.public.apiBase}/blog/posts/paginated?page=${page}&per_page=${pageSize}`
    )

    posts.value = response.data
    totalPosts.value = response.meta.total
    totalPages.value = response.meta.last_page
  } catch (error) {
    console.error('Помилка завантаження:', error)
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  getPosts(1)
})

const columns = [
  {
    accessorKey: 'id',
    header: '#',
    cell: ({ row }) =>
        h('div', { class: 'flex items-center gap-3' }, [
          h('span', row.original.id),
          h(
              NuxtLink,
              {
                to: `/blog/${row.original.id}`,
                class: 'px-2 py-1 bg-blue-600 text-white rounded hover:bg-blue-700 transition-colors text-xs',
              },
              () => '→'
          ),
        ]),
  },
  {
    accessorKey: 'title',
    header: 'Заголовок',
  },
  {
    accessorKey: 'user.name',
    header: 'Автор',
  },
  {
    accessorKey: 'category.title',
    header: 'Категорія',
  },
]
</script>

<style scoped>
:deep(tbody tr) {
  cursor: pointer;
}

:deep(tbody tr:hover) {
  background-color: rgba(59, 130, 246, 0.1);
}
</style>
