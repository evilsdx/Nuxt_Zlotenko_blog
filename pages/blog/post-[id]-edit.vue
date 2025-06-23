<template>
  <div class="min-h-screen bg-black p-10 text-white flex justify-center">
    <div class="w-full max-w-3xl space-y-10">
      <h1 class="text-4xl font-bold text-center mb-6">Редагувати пост</h1>

      <div v-if="loading" class="text-center text-gray-400 text-lg">Завантаження...</div>

      <form
          v-else-if="post"
          @submit.prevent="updatePost"
          class="bg-gray-900 p-8 rounded-lg space-y-6 shadow-lg"
      >
        <UFormGroup label="Заголовок" class="font-semibold">
          <UInput
              v-model="post.title"
              placeholder="Введіть заголовок посту"
              required
              autocomplete="off"
          />
        </UFormGroup>

        <UFormGroup label="Псевдонім (slug)" class="font-semibold">
          <UInput
              v-model="post.slug"
              placeholder="Унікальний псевдонім для URL (необов’язково)"
              autocomplete="off"
          />
          <p class="text-gray-400 text-xs mt-1">Якщо залишити порожнім, буде згенеровано автоматично</p>
        </UFormGroup>

        <UFormGroup label="Короткий опис" class="font-semibold">
          <UTextarea
              v-model="post.excerpt"
              rows="3"
              placeholder="Короткий опис або анотація посту"
          />
        </UFormGroup>

        <UFormGroup label="Контент" class="font-semibold">
          <UTextarea
              v-model="post.content_raw"
              class="min-h-[150px]"
              placeholder="Повний текст посту"
              required
          />
        </UFormGroup>

        <UFormGroup label="ID Категорії" class="font-semibold">
          <UInput
              v-model.number="post.category_id"
              type="number"
              placeholder="Введіть ID категорії (наприклад, 1)"
              required
              min="1"
          />
        </UFormGroup>

        <UFormGroup label="Опублікувати" class="font-semibold flex items-center space-x-3">
          <UToggle v-model="post.is_published" />
          <span>Опубліковано</span>
        </UFormGroup>

        <div v-if="errorMessage" class="bg-red-800 text-red-300 p-3 rounded">
          Помилка: {{ errorMessage }}
        </div>

        <div class="flex justify-between items-center">
          <NuxtLink
              to="/BlogPostsUi"
              class="text-gray-400 hover:text-white underline"
          >
            ← Назад
          </NuxtLink>
          <UButton
              type="submit"
              label="Оновити"
              color="primary"
              :loading="loading"
              class="min-w-[120px]"
          />
        </div>
      </form>

      <div v-else class="text-center text-red-400 text-lg">Пост не знайдено</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute, navigateTo } from '#imports'

const config = useRuntimeConfig()
const route = useRoute()

const post = ref<PostForm | null>(null)
const loading = ref(false)
const errorMessage = ref('')

interface PostForm {
  title: string
  slug: string
  excerpt: string
  content_raw: string
  is_published: boolean
  category_id: number | null
  user_id: number
}

const getPost = async () => {
  loading.value = true
  errorMessage.value = ''
  try {
    const id = route.params.id
    const response = await $fetch<any>(`${config.public.apiBase}/blog/posts/${id}`)

    post.value = {
      title: response.title || '',
      slug: response.slug || '',
      excerpt: response.excerpt || '',
      content_raw: response.content_raw || '',
      is_published: response.is_published || false,
      category_id: response.category?.id ?? null,
      user_id: response.user?.id ?? 1
    }
  } catch (error) {
    console.error('Помилка отримання посту:', error)
    post.value = null
    errorMessage.value = 'Не вдалося завантажити пост'
  } finally {
    loading.value = false
  }
}

const updatePost = async () => {
  if (!post.value) return
  loading.value = true
  errorMessage.value = ''
  try {
    const id = route.params.id

    await $fetch(`${config.public.apiBase}/blog/posts/${id}`, {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(post.value)
    })

    await navigateTo('/BlogPostsUi')
  } catch (error: any) {
    console.error('Помилка оновлення посту:', error)
    if (error?.data?.message) {
      errorMessage.value = error.data.message
    } else if (error?.message) {
      errorMessage.value = error.message
    } else {
      errorMessage.value = 'Сталася невідома помилка при оновленні'
    }
  } finally {
    loading.value = false
  }
}

onMounted(() => {
  getPost()
})
</script>
