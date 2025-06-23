<template>
  <div class="min-h-screen bg-black p-10 text-white flex justify-center">
    <div class="w-full max-w-3xl space-y-10">
      <h1 class="text-4xl font-bold text-center mb-6">Створити новий пост</h1>

      <form @submit.prevent="createPost" class="bg-gray-900 p-8 rounded-lg space-y-6 shadow-lg">
        <UFormGroup label="Заголовок" class="font-semibold">
          <UInput
              v-model="form.title"
              placeholder="Введіть заголовок посту"
              required
              autocomplete="off"
          />
        </UFormGroup>

        <UFormGroup label="Псевдонім (slug)" class="font-semibold">
          <UInput
              v-model="form.slug"
              placeholder="Унікальний псевдонім для URL (необов’язково)"
              autocomplete="off"
          />
          <p class="text-gray-400 text-xs mt-1">Якщо залишити порожнім, буде згенеровано автоматично</p>
        </UFormGroup>

        <UFormGroup label="Короткий опис" class="font-semibold">
          <UTextarea
              v-model="form.excerpt"
              placeholder="Короткий опис або анотація посту"
              rows="3"
          />
        </UFormGroup>

        <UFormGroup label="Контент" class="font-semibold">
          <UTextarea
              v-model="form.content_raw"
              placeholder="Повний текст посту"
              rows="8"
              required
          />
        </UFormGroup>

        <UFormGroup label="ID Категорії" class="font-semibold">
          <UInput
              v-model.number="form.category_id"
              type="number"
              placeholder="Введіть ID категорії (наприклад, 1)"
              required
              min="1"
          />
        </UFormGroup>

        <UFormGroup label="Опублікувати" class="font-semibold flex items-center space-x-3">
          <UToggle v-model="form.is_published" />
          <span class="text-gray-300">Опублікувати зараз</span>
        </UFormGroup>

        <div v-if="errorMessage" class="bg-red-800 text-red-300 p-3 rounded">
          Помилка: {{ errorMessage }}
        </div>

        <UButton
            type="submit"
            label="Створити"
            color="primary"
            :loading="loading"
            class="w-full"
        />
      </form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { reactive, ref } from 'vue'
import { useRuntimeConfig, navigateTo } from '#imports'

const config = useRuntimeConfig()

const form = reactive({
  title: '',
  slug: '',
  excerpt: '',
  content_raw: '',
  is_published: false,
  category_id: null as number | null,
  user_id: 1
})

const loading = ref(false)
const errorMessage = ref('')

const createPost = async () => {
  errorMessage.value = ''
  loading.value = true

  try {
    await $fetch(`${config.public.apiBase}/blog/posts`, {
      method: 'POST',
      body: form
    })
    await navigateTo('/BlogPostsUi')
  } catch (error: any) {
    if (error?.data?.message) {
      errorMessage.value = error.data.message
    } else if (error?.message) {
      errorMessage.value = error.message
    } else {
      errorMessage.value = 'Сталася невідома помилка'
    }
    console.error(error)
  } finally {
    loading.value = false
  }
}
</script>
