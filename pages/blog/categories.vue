<script setup lang="ts">
import type { TableColumn } from '@nuxt/ui'
import { h, resolveComponent, computed } from 'vue'
import { navigateTo } from '#app'

interface Category {
  id: number
  title: string
  slug: string
  parent_id: number
  description: string | null
}

const config = useRuntimeConfig()

const { data: categories, error, refresh } = await useFetch<{ data: Category[] }>(
    `${config.public.apiBase}/blog/categories`
)

const deleteCategory = async (id: number) => {
  if (!confirm(`Ви впевнені, що хочете видалити категорію з ID ${id}?`)) return

  try {
    await $fetch(`${config.public.apiBase}/blog/categories/${id}`, {
      method: 'DELETE',
    })
    await refresh()
  } catch (err) {
    console.error('Помилка видалення:', err)
    alert('Не вдалося видалити категорію')
  }
}

const UDropdownMenu = resolveComponent('UDropdownMenu')
const UButton = resolveComponent('UButton')

const columns: TableColumn<Category>[] = [
  { accessorKey: 'id', header: '#' },
  { accessorKey: 'slug', header: 'Slug' },
  { accessorKey: 'title', header: 'Назва' },
  {
    accessorKey: 'parent_id',
    header: 'Батьківська категорія',
    cell: ({ row }) => (row.original.parent_id === 0 ? 'Корінь' : String(row.original.parent_id)),
  },
  { accessorKey: 'description', header: 'Опис' },
  {
    accessorKey: 'actions',
    header: 'Дії',
    cell: ({ row }) =>
        h(
            UDropdownMenu,
            {
              items: [
                {
                  label: 'Редагувати',
                  icon: 'i-heroicons-pencil-square',
                  onClick: () => navigateTo(`/blog/categories-${row.original.id}-edit`),
                },
                {
                  label: 'Видалити',
                  icon: 'i-heroicons-trash',
                  onClick: () => deleteCategory(row.original.id),
                  color: 'danger',
                },
              ],
              'aria-label': 'Меню дій',
              class: 'inline-block',
              contentClass: 'min-w-[120px]',
              placement: 'bottom-end',
            },
            {
              default: () =>
                  h(UButton, {
                    icon: 'i-heroicons-ellipsis-vertical',
                    variant: 'ghost',
                    size: 'sm',
                    color: 'neutral',
                    'aria-label': 'Відкрити меню дій',
                    class: 'hover:bg-gray-100 dark:hover:bg-gray-700',
                  }),
            }
        ),
  },
]

const categoryCount = computed(() => categories.value?.data.length || 0)
</script>

<template>
  <div class="w-full px-8 max-w-7xl mx-auto py-12 font-sans bg-white dark:bg-gray-900 text-gray-900 dark:text-gray-100">
    <div class="flex justify-between items-center mb-6">
      <div>
        <h1 class="text-4xl font-semibold tracking-tight">Категорії</h1>
        <p class="text-sm text-gray-500 dark:text-gray-400 mt-1">
          Усього знайдено: {{ categoryCount }} {{ categoryCount === 1 ? 'категорія' : 'категорій' }}
        </p>
      </div>
      <NuxtLink
          to="/blog/categories-create"
          class="px-5 py-2 bg-[#00C58E] hover:bg-[#00a974] text-white rounded-md shadow transition"
      >
        + Створити
      </NuxtLink>
    </div>

    <div v-if="error" class="text-red-600 font-semibold mb-6">Помилка завантаження категорій</div>

    <UTable
        v-else-if="categories && categories.data.length"
        :columns="columns"
        :data="categories.data"
        class="bg-gray-50 dark:bg-gray-800 rounded-md shadow border border-gray-200 dark:border-gray-700 overflow-hidden"
    />

    <p v-else class="text-gray-500 dark:text-gray-400 text-center py-12 text-lg">
      Категорії відсутні
    </p>
  </div>
</template>
