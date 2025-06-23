<script setup lang="ts">
import { ref } from 'vue'
import { useRouter, useRuntimeConfig } from '#imports'

const config = useRuntimeConfig()
const router = useRouter()

const form = ref({
  title: '',
  slug: '',
  description: '',
  parent_id: 0
})

const submitting = ref(false)
const errorMsg = ref('')

const handleSubmit = async () => {
  submitting.value = true
  errorMsg.value = ''

  try {
    const data = { ...form.value }
    if (!data.slug) data.slug = data.title.toLowerCase().replace(/\s+/g, '-')

    await $fetch(`${config.public.apiBase}/blog/categories`, {
      method: 'POST',
      body: data
    })

    router.push('/blog/categories')
  } catch (e) {
    errorMsg.value = 'Помилка створення категорії'
  } finally {
    submitting.value = false
  }
}
</script>

<template>
  <div class="max-w-3xl mx-auto py-12 px-4">
    <h1 class="text-3xl font-bold mb-6">Створити категорію</h1>
    <form @submit.prevent="handleSubmit" class="space-y-4">
      <div>
        <label class="block text-sm font-medium">Назва</label>
        <input v-model="form.title" type="text" class="w-full rounded border px-3 py-2" required />
      </div>

      <div>
        <label class="block text-sm font-medium">Slug</label>
        <input v-model="form.slug" type="text" class="w-full rounded border px-3 py-2" />
      </div>

      <div>
        <label class="block text-sm font-medium">Опис</label>
        <textarea v-model="form.description" rows="3" class="w-full rounded border px-3 py-2"></textarea>
      </div>

      <div>
        <label class="block text-sm font-medium">Батьківська категорія (ID)</label>
        <input v-model.number="form.parent_id" type="number" class="w-full rounded border px-3 py-2" min="0" required />
      </div>

      <div class="pt-4">
        <button type="submit" class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded" :disabled="submitting">
          {{ submitting ? 'Збереження...' : 'Зберегти' }}
        </button>
        <span v-if="errorMsg" class="text-red-600 ml-4">{{ errorMsg }}</span>
      </div>
    </form>
  </div>
</template>
