<template>
  <div class="min-h-screen bg-black p-10 text-white">
    <div class="max-w-6xl mx-auto mt-20 space-y-10">
      <div class="text-center">
        <h1 class="text-3xl font-semibold">Список постів блогу</h1>
      </div>

      <div class="flex justify-between items-center">
        <div class="text-gray-400 text-sm">Всього постів: {{ totalPosts }}</div>
        <UButton
            label="Створити пост"
            icon="i-heroicons-plus"
            color="primary"
            @click="navigateTo('/blog/post-create')"
        />
      </div>

      <div class="w-full space-y-4 pb-4">
        <UTable
            :data="posts"
            :columns="columns"
            :loading="loading"
            class="flex-1 bg-gray-900 border border-gray-700 rounded-lg"
        >
          <template #actions-cell="{ row }">
            <UDropdownMenu
                :items="getDropdownActions(row.original)"
                placement="bottom-end"
                class="inline-block"
                @click.stop
            >
              <UButton
                  icon="i-heroicons-ellipsis-vertical"
                  color="neutral"
                  variant="ghost"
                  aria-label="Дії"
                  @click.stop
              />
            </UDropdownMenu>
          </template>
        </UTable>

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
import type { TableColumn, DropdownMenuItem } from '@nuxt/ui'
import { h } from 'vue'
import { navigateTo } from '#app'

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

onMounted(() => {
  getPosts(1)
})

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

const goToPage = async (page: number) => {
  if (page < 1 || page > totalPages.value || page === currentPage.value) return
  currentPage.value = page
  await getPosts(page)
}

const goToBlog = (post: Post) => {
  navigateTo(`/blog/${post.id}`)
}

const deletePost = async (id: number) => {
  if (!confirm(`Ви впевнені, що хочете видалити пост з ID ${id}?`)) return
  try {
    await $fetch(`${config.public.apiBase}/blog/posts/${id}`, {
      method: 'DELETE'
    })
    await getPosts(currentPage.value)
  } catch (error) {
    alert('Не вдалося видалити пост')
    console.error('Помилка видалення:', error)
  }
}

function getDropdownActions(post: Post): DropdownMenuItem[][] {
  return [
    [
      {
        label: 'Редагувати',
        icon: 'i-heroicons-pencil-square',
        onSelect: () => navigateTo(`/blog/post-${post.id}-edit`)
      },
      {
        label: 'Видалити',
        icon: 'i-heroicons-trash',
        color: 'error',
        onSelect: () => deletePost(post.id)
      }
    ]
  ]
}

const columns: TableColumn<Post>[] = [
  {
    accessorKey: 'id',
    header: '#'
  },
  {
    accessorKey: 'title',
    header: 'Заголовок',
    cell: ({ row }) =>
        h(
            'a',
            {
              class: 'text-blue-400 hover:underline cursor-pointer',
              onClick: (e: MouseEvent) => {
                e.stopPropagation()
                goToBlog(row.original)
              }
            },
            row.original.title
        )
  },
  {
    accessorKey: 'user.name',
    header: 'Автор'
  },
  {
    accessorKey: 'category.title',
    header: 'Категорія'
  },
  {
    id: 'actions',
    header: 'Дії'
  }
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
