<template>
  <div class="min-h-screen bg-black p-10 text-white">
    <div class="max-w-6xl mx-auto mt-20 space-y-10">
      <NuxtLink
          to="/BlogPostsUi"
          class="text-white hover:text-gray-300 transition-colors cursor-pointer"
      >
        ← Назад до списку
      </NuxtLink>

      <div v-if="loading" class="text-center">
        <div class="text-gray-400">Завантаження...</div>
      </div>

      <div v-else-if="post" class="grid grid-cols-1 lg:grid-cols-4 gap-6">
        <div class="lg:col-span-3 space-y-6">
          <div class="bg-gray-900 rounded-lg p-4">
            <div class="text-lg font-semibold mb-2">
              <span v-if="post.is_published" class="text-green-400">Опубліковано</span>
              <span v-else class="text-yellow-400">Чернетка</span>
            </div>
          </div>

          <div class="bg-gray-900 rounded-lg p-6">
            <h2 class="text-xl font-semibold mb-4 text-blue-400">Основні дані</h2>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Заголовок</label>
              <div class="bg-gray-800 p-3 rounded border text-white">
                {{ post.title }}
              </div>
            </div>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Текст статті</label>
              <div class="bg-gray-800 p-4 rounded border text-gray-300 min-h-[200px] whitespace-pre-wrap">
                {{ post.content_raw }}
              </div>
            </div>
          </div>

          <div class="bg-gray-900 rounded-lg p-6">
            <h2 class="text-xl font-semibold mb-4 text-blue-400">Додаткові дані</h2>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Категорія</label>
              <div class="bg-gray-800 p-3 rounded border text-white">
                {{ post.category?.title || 'Без категорії' }}
              </div>
            </div>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Псевдонім</label>
              <div class="bg-gray-800 p-3 rounded border text-white">
                {{ post.slug || 'Не вказано' }}
              </div>
            </div>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Короткий текст</label>
              <div class="bg-gray-800 p-3 rounded border text-gray-300 min-h-[80px]">
                {{ post.excerpt || 'Не вказано' }}
              </div>
            </div>

            <div class="mb-4">
              <label class="block text-sm font-medium text-gray-300 mb-2">Статус</label>
              <div class="flex items-center">
                <input
                    type="checkbox"
                    :checked="post.is_published"
                    disabled
                    class="mr-2"
                >
                <span class="text-white">Опубліковано</span>
              </div>
            </div>
          </div>
        </div>

        <div class="lg:col-span-1 space-y-6">
          <div class="bg-gray-900 rounded-lg p-4">
            <div class="text-sm text-gray-300">ID: {{ post.id }}</div>
          </div>

          <div class="bg-gray-900 rounded-lg p-4 space-y-4">
            <div>
              <label class="block text-sm font-medium text-gray-300 mb-1">Створено</label>
              <div class="bg-gray-800 p-2 rounded text-sm text-white">
                {{ formatDate(post.created_at) }}
              </div>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-300 mb-1">Змінено</label>
              <div class="bg-gray-800 p-2 rounded text-sm text-white">
                {{ formatDate(post.updated_at) }}
              </div>
            </div>

            <div>
              <label class="block text-sm font-medium text-gray-300 mb-1">Опубліковано</label>
              <div class="bg-gray-800 p-2 rounded text-sm text-white">
                {{ post.published_at ? formatDate(post.published_at) : 'Не опубліковано' }}
              </div>
            </div>
          </div>

          <div class="bg-gray-900 rounded-lg p-4">
            <label class="block text-sm font-medium text-gray-300 mb-1">Автор</label>
            <div class="text-white">{{ post.user?.name || 'Невідомо' }}</div>
          </div>

          <div class="bg-gray-900 rounded-lg p-4">
            <NuxtLink
                :to="`/blog/post-${post.id}-edit`"
                class="text-white hover:text-gray-300 transition-colors cursor-pointer block text-center"
            >
              Редагувати
            </NuxtLink>
          </div>
        </div>
      </div>

      <div v-else class="text-center">
        <div class="text-red-400">Пост не знайдено</div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
const route = useRoute()
const config = useRuntimeConfig()

interface Post {
  id: number
  title: string
  slug: string
  excerpt: string
  content_raw: string
  content: string
  is_published: boolean
  published_at: string | null
  created_at: string
  updated_at: string
  user: {
    id: number
    name: string
  } | null
  category: {
    id: number
    title: string
  } | null
}

const post = ref<Post | null>(null)
const loading = ref(true)

const formatDate = (dateString: string) => {
  if (!dateString) return ''
  return new Date(dateString).toLocaleString('uk-UA', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit'
  })
}

const getPost = async () => {
  try {
    const postId = route.params.id
    const response = await $fetch<Post>(`${config.public.apiBase}/blog/posts/${postId}`)
    post.value = response
    console.log('Отримані дані посту:', response)
  } catch (error) {
    console.error('Помилка завантаження посту:', error)
    post.value = null
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  getPost()
})

useHead({
  title: computed(() => post.value ? post.value.title : 'Завантаження...'),
  meta: [
    {
      name: 'description',
      content: computed(() => post.value ? `${post.value.title} - ${post.value.user?.name || ''}` : '')
    }
  ]
})
</script>