<script setup lang="ts">
import { h, ref, computed } from 'vue'
import type { TableColumn } from '@nuxt/ui'
import { getPaginationRowModel } from '@tanstack/vue-table'

// Базові налаштування
useHead({ title: 'Каталог продуктів' })
const table = useTemplateRef('table')

// Типи даних
type Product = {
  id: number
  title: string
  description: string
  price: number
  rating: number
  category: string
  thumbnail: string
}

type ApiResponse = {
  products: Product[]
  total: number
  skip: number
  limit: number
}

// Стан застосунку
const searchQuery = ref('')
const pagination = ref({ pageIndex: 0, pageSize: 8 })

// Використання useFetch для отримання даних
const { data, pending, error } = useFetch<ApiResponse>('https://dummyjson.com/products', {
  query: {
    limit: 100
  },
  key: 'products',
  transform: (response) => response
})

// Доступ до масиву products або порожнього масиву якщо дані ще не завантажені
const products = computed(() => data.value?.products || [])

// Фільтрація даних
const globalFilteredData = computed(() => {
  if (!searchQuery.value.trim()) return products.value

  const searchLower = searchQuery.value.toLowerCase().trim()
  return products.value.filter(product =>
      product.title?.toLowerCase().includes(searchLower) ||
      product.description?.toLowerCase().includes(searchLower) ||
      product.category?.toLowerCase().includes(searchLower)
  )
})

// Функції управління
function updateFilter(value: string | number) {
  if (typeof value === 'string') {
    searchQuery.value = value
    table?.value?.tableApi?.setPageIndex(0)
  }
}

// Визначення колонок таблиці
const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    cell: ({ row }) => h('img', {
      src: row.getValue('thumbnail'),
      alt: 'Продукт',
      class: 'h-16 w-16 rounded object-cover border border-purple-100'
    }),
    enableSorting: false,
  },
  {
    accessorKey: 'title',
    header: ({ column }) => h('div', {
      class: 'cursor-pointer select-none',
      onClick: () => column.toggleSorting()
    }, [
      'Назва',
      column.getIsSorted() === 'asc' ? ' ↑' : column.getIsSorted() === 'desc' ? ' ↓' : ''
    ]),
    cell: ({ row }) => h('div', { class: 'font-medium' }, row.getValue('title')),
    enableSorting: true,
  },
  {
    accessorKey: 'price',
    header: ({ column }) => h('div', {
      class: 'cursor-pointer select-none',
      onClick: () => column.toggleSorting()
    }, [
      'Ціна',
      column.getIsSorted() === 'asc' ? ' ↑' : column.getIsSorted() === 'desc' ? ' ↓' : ''
    ]),
    cell: ({ row }) => {
      const price = Number(row.getValue('price'))
      return h('div', { class: 'font-medium' },
          `${price.toFixed(2)} $`
      )
    },
    enableSorting: true,
  },
  {
    accessorKey: 'rating',
    header: ({ column }) => h('div', {
      class: 'cursor-pointer select-none',
      onClick: () => column.toggleSorting()
    }, [
      'Рейтинг',
      column.getIsSorted() === 'asc' ? ' ↑' : column.getIsSorted() === 'desc' ? ' ↓' : ''
    ]),
    cell: ({ row }) => {
      const rating = Number(row.getValue('rating'))
      const stars = '★'.repeat(Math.floor(rating)) + '☆'.repeat(5 - Math.floor(rating))
      return h('div', {
        class: `font-medium ${rating >= 4.5 ? 'text-green-500' : 'text-red-500'}`
      }, [
        `${rating.toFixed(1)} `, // Display numeric rating with 1 decimal place
        stars
      ])
    },
    enableSorting: true,
  },
  {
    accessorKey: 'category',
    header: 'Категорія',
    cell: ({ row }) => h('span', {
      class: 'px-2 py-1 rounded bg-purple-100 text-purple-800 text-xs font-medium'
    }, row.getValue('category')),
    enableSorting: true,
  }
]
</script>

<template>
  <div class="min-h-screen w-full bg-gradient-to-br from-indigo-50 via-purple-50 to-pink-50 dark:from-gray-900 dark:via-indigo-950 dark:to-purple-900 text-gray-800 dark:text-gray-200 transition-colors duration-500">
    <!-- Futuristic Hero section -->
    <div class="relative overflow-hidden bg-white/80 dark:bg-gray-900/80 backdrop-blur-lg p-8 rounded-2xl mx-4 mt-6 shadow-xl border-b-4 border-indigo-400 dark:border-indigo-600">
      <div class="absolute -top-24 -right-24 w-48 h-48 bg-gradient-to-br from-purple-400 to-pink-400 rounded-full opacity-20 blur-xl"></div>
      <div class="absolute -bottom-12 -left-12 w-36 h-36 bg-gradient-to-tr from-indigo-400 to-blue-400 rounded-full opacity-20 blur-xl"></div>

      <h1 class="text-5xl font-black tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-indigo-600 via-purple-500 to-pink-500 dark:from-indigo-400 dark:via-purple-300 dark:to-pink-300 relative z-10">
        <span class="inline-block transform hover:scale-105 transition-transform duration-300"></span> Каталог продуктів
      </h1>
      <p class="text-md text-gray-600 dark:text-gray-400 mt-2 font-medium max-w-2xl relative z-10">
        Лабораторна робота #8 з використанням <strong>Vue 3</strong>, <strong>Nuxt 3</strong> та <strong>DummyJSON</strong>.
      </p>
    </div>

    <!-- Floating loading bar -->
    <div v-if="pending" class="w-full h-2 fixed top-0 left-0 z-50 bg-gray-200/50 dark:bg-gray-800/50">
      <div class="h-2 bg-gradient-to-r from-indigo-500 via-purple-500 to-pink-500 transition-all duration-500 ease-out animate-pulse" style="width: 100%"></div>
    </div>

    <!-- Search Controls with glow effect -->
    <div class="flex flex-wrap gap-4 items-center justify-between p-5 mx-4 mt-6 bg-white/70 dark:bg-gray-900/70 backdrop-blur-lg rounded-2xl shadow-lg border-l-4 border-indigo-400 dark:border-indigo-600 relative overflow-hidden">
      <div class="absolute -bottom-16 -right-16 w-32 h-32 bg-indigo-300 dark:bg-indigo-700 rounded-full opacity-20 blur-xl"></div>

      <div class="flex gap-4 items-center relative z-10 w-full sm:w-auto">
        <UInput
            v-model="searchQuery"
            placeholder="Пошук продуктів..."
            icon="i-heroicons-magnifying-glass"
            class="w-full sm:w-96 bg-white dark:bg-gray-800 border border-indigo-200 dark:border-indigo-800 rounded-xl shadow-inner focus:ring-2 focus:ring-indigo-400"
            @update:model-value="updateFilter"
        />
      </div>
    </div>

    <!-- Loading state -->
    <div v-if="pending" class="flex items-center justify-center h-screen">
      <UButton
          variant="solid"
          size="lg"
          class="animate-pulse"
          :disabled="true"
      >
        Завантаження продуктів...
      </UButton>
    </div>

    <!-- Error message -->
    <div v-if="error" class="mx-4 mt-6 p-4 bg-red-100 dark:bg-red-900/30 text-red-700 dark:text-red-300 rounded-xl flex items-center gap-2">
      <div class="w-6 h-6 rounded-full bg-red-200 dark:bg-red-800 flex items-center justify-center flex-shrink-0">
        <span class="text-red-600 dark:text-red-300">!</span>
      </div>
      Не вдалося завантажити продукти. Спробуйте ще раз.
    </div>

    <!-- Table with glass morphism effect -->
    <div v-else-if="!pending" class="px-6 py-6 mx-4 mt-6">
      <UTable
          ref="table"
          v-model:pagination="pagination"
          :data="globalFilteredData"
          :columns="columns"
          sticky
          hover
          :pagination-options="{
          getPaginationRowModel: getPaginationRowModel()
        }"
          class="rounded-2xl bg-white/90 dark:bg-gray-900/90 backdrop-blur-lg shadow-xl border-t-4 border-indigo-400 dark:border-indigo-600 overflow-hidden"
          :ui="{
          thead: 'bg-indigo-50 dark:bg-indigo-950/50',
          tbody: 'divide-y divide-indigo-100 dark:divide-indigo-900',
          tr: 'transition-colors hover:bg-indigo-50/50 dark:hover:bg-indigo-950/50'
        }"
      />
    </div>

    <!-- Modern pagination control -->
    <div v-if="!pending && !error" class="flex justify-center items-center px-6 py-5 mx-4 mt-6 mb-6 bg-white/70 dark:bg-gray-900/70 backdrop-blur-lg rounded-2xl shadow-lg border-r-4 border-indigo-400 dark:border-indigo-600 relative overflow-hidden">
      <div class="absolute -top-16 -left-16 w-32 h-32 bg-purple-300 dark:bg-purple-700 rounded-full opacity-20 blur-xl"></div>

      <UPagination
          :default-page="(table?.tableApi?.getState().pagination.pageIndex || 0) + 1"
          :items-per-page="table?.tableApi?.getState().pagination.pageSize"
          :total="table?.tableApi?.getFilteredRowModel().rows.length || 0"
          @update:page="(p) => table?.tableApi?.setPageIndex(p - 1)"
      />
    </div>
  </div>
</template>

<style scoped>
button {
  transition: all 0.3s ease-in-out;
}
button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
button:not(:disabled):hover {
  transform: translateY(-2px);
}
</style>
