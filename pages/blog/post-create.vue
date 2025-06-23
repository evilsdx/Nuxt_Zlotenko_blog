<script setup lang="ts">
import { reactive, ref } from 'vue'
import { useRuntimeConfig, navigateTo } from '#imports'
import * as z from 'zod'
import type { FormSubmitEvent } from '@nuxt/ui'

const schema = z.object({
  title: z.string().min(3, 'Заголовок має бути не менше 3 символів'),
  slug: z.string().optional(),
  excerpt: z.string().optional(),
  content_raw: z.string().min(10, 'Контент має бути не менше 10 символів'),
  is_published: z.boolean(),
  category_id: z.number({ invalid_type_error: 'Вкажіть ID категорії' }).min(1, 'ID має бути більше 0'),
  user_id: z.number()
})

type Schema = z.output<typeof schema>

const config = useRuntimeConfig()

const form = reactive<Partial<Schema>>({
  title: '',
  slug: '',
  excerpt: '',
  content_raw: '',
  is_published: false,
  category_id: null,
  user_id: 1
})

const loading = ref(false)
const errorMessage = ref('')

const createPost = async (event: FormSubmitEvent<Schema>) => {
  errorMessage.value = ''
  loading.value = true

  try {
    const data = { ...event.data }

    if (!data.slug) {
      data.slug = data.title.toLowerCase().replace(/\s+/g, '-').replace(/[^a-z0-9\-]/g, '')
    }

    await $fetch(`${config.public.apiBase}/blog/posts`, {
      method: 'POST',
      body: data
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
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-black text-white flex items-center justify-center">
    <div class="w-full max-w-3xl space-y-10 p-10">
      <h1 class="text-4xl font-bold text-center mb-6">Створити новий пост</h1>

      <UForm
          :state="form"
          :schema="schema"
          @submit="createPost"
          class="bg-gray-900 p-8 rounded-lg space-y-6 shadow-lg"
      >
        <UFormField label="Заголовок" name="title">
          <UInput v-model="form.title" placeholder="Введіть заголовок посту" />
        </UFormField>

        <UFormField label="Псевдонім (slug)" name="slug">
          <UInput v-model="form.slug" placeholder="Унікальний псевдонім для URL (необов’язково)" />
        </UFormField>

        <UFormField label="Короткий опис" name="excerpt">
          <UTextarea v-model="form.excerpt" placeholder="Короткий опис або анотація посту" rows="3" />
        </UFormField>

        <UFormField label="Контент" name="content_raw">
          <UTextarea v-model="form.content_raw" placeholder="Повний текст посту" rows="8" />
        </UFormField>

        <UFormField label="ID Категорії" name="category_id">
          <UInput v-model.number="form.category_id" type="number" placeholder="ID категорії" min="1" />
        </UFormField>

        <UFormField name="is_published">
          <template #label>
            <span class="font-semibold flex items-center space-x-3">
              <span>Опублікувати зараз</span>
            </span>
          </template>
          <UToggle v-model="form.is_published" />
        </UFormField>

        <div v-if="errorMessage" class="bg-red-800 text-red-300 p-3 rounded">
          Помилка: {{ errorMessage }}
        </div>

        <UButton type="submit" label="Створити" color="primary" :loading="loading" class="w-full" />
      </UForm>
    </div>
  </div>
</template>
