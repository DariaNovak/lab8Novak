<template>
  <div class="container">
    <div class="content">
      <header class="header">
        <h1 class="title">Продукти</h1>
        <p class="subtitle">Перегляд та управління каталогом продуктів</p>
      </header>

      <!-- Search and filters -->
      <div class="card search-card">
        <div class="search-container">
          <div class="search-icon">
            <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
              <circle cx="11" cy="11" r="8"></circle>
              <line x1="21" y1="21" x2="16.65" y2="16.65"></line>
            </svg>
          </div>
          <input
              v-model="search"
              placeholder="Пошук продуктів..."
              class="search-input"
          />
        </div>
      </div>

      <!-- Loading state -->
      <div v-if="pending" class="loading-container">
        <div class="loading-spinner"></div>
        <p>Завантаження продуктів...</p>
      </div>

      <!-- Table with data -->
      <div v-else class="card table-card">
        <div class="table-container">
          <table class="products-table">
            <thead>
            <tr>
              <th
                  v-for="col in columns"
                  :key="col.key"
                  class="table-header"
                  :class="{'sortable': col.sortable}"
                  @click="col.sortable ? updateSort(col.key) : null"
              >
                <div class="header-content">
                  <span>{{ col.label }}</span>
                  <span v-if="col.sortable" class="sort-icon">
                      <svg v-if="sort.column === col.key && sort.direction === 'asc'" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <polyline points="18 15 12 9 6 15"></polyline>
                      </svg>
                      <svg v-else-if="sort.column === col.key && sort.direction === 'desc'" xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                        <polyline points="6 9 12 15 18 9"></polyline>
                      </svg>
                      <svg v-else xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="sort-default">
                        <line x1="12" y1="5" x2="12" y2="19"></line>
                        <polyline points="19 12 12 19 5 12"></polyline>
                      </svg>
                    </span>
                </div>
              </th>
            </tr>
            </thead>
            <tbody>
            <tr v-for="product in paginatedProducts" :key="product.id" class="table-row">
              <td class="table-cell">{{ product.title }}</td>
              <td class="table-cell description-cell">
                <div class="truncated" :title="product.description">
                  {{ product.description }}
                </div>
              </td>
              <td class="table-cell price-cell">${{ product.price.toFixed(2) }}</td>
              <td class="table-cell rating-cell">
                <div class="rating" :class="product.rating >= 4.5 ? 'rating-high' : 'rating-low'">
                  {{ product.rating.toFixed(1) }}
                  <span class="rating-star">★</span>
                </div>
              </td>
              <td class="table-cell">{{ product.brand }}</td>
              <td class="table-cell">
                <span class="category-tag">{{ product.category }}</span>
              </td>
              <td class="table-cell image-cell">
                <img
                    :src="product.thumbnail"
                    :alt="product.title"
                    class="product-image"
                />
              </td>
            </tr>
            <tr v-if="paginatedProducts.length === 0">
              <td colspan="7" class="empty-message">
                <div class="empty-content">
                  <svg xmlns="http://www.w3.org/2000/svg" width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1" stroke-linecap="round" stroke-linejoin="round">
                    <circle cx="12" cy="12" r="10"></circle>
                    <path d="M8 15h8"></path>
                    <path d="M9 9h.01"></path>
                    <path d="M15 9h.01"></path>
                  </svg>
                  <p>Продукти не знайдено</p>
                  <p class="empty-hint">Спробуйте змінити параметри пошуку</p>
                </div>
              </td>
            </tr>
            </tbody>
          </table>
        </div>

        <!-- Table footer with pagination -->
        <div class="table-footer">
          <div class="per-page-selector">
            <label for="per-page" class="per-page-label">Показувати:</label>
            <select id="per-page" v-model="perPage" class="per-page-select">
              <option :value="5">5</option>
              <option :value="10">10</option>
              <option :value="20">20</option>
              <option :value="50">50</option>
            </select>
          </div>

          <div class="pagination">
            <button
                @click="page = 1"
                :disabled="page === 1"
                class="page-button page-first"
                title="Перша сторінка"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="11 17 6 12 11 7"></polyline>
                <polyline points="18 17 13 12 18 7"></polyline>
              </svg>
            </button>
            <button
                @click="page--"
                :disabled="page === 1"
                class="page-button page-prev"
                title="Попередня сторінка"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="15 18 9 12 15 6"></polyline>
              </svg>
            </button>

            <div class="page-numbers">
              <button
                  v-for="pageNum in displayedPages"
                  :key="pageNum"
                  @click="page = pageNum"
                  class="page-button page-number"
                  :class="{'active': page === pageNum}"
              >
                {{ pageNum }}
              </button>
            </div>

            <button
                @click="page++"
                :disabled="page === totalPages"
                class="page-button page-next"
                title="Наступна сторінка"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="9 18 15 12 9 6"></polyline>
              </svg>
            </button>
            <button
                @click="page = totalPages"
                :disabled="page === totalPages"
                class="page-button page-last"
                title="Остання сторінка"
            >
              <svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round">
                <polyline points="13 17 18 12 13 7"></polyline>
                <polyline points="6 17 11 12 6 7"></polyline>
              </svg>
            </button>
          </div>

          <div class="status-info">
            {{ (page - 1) * perPage + 1 }}-{{ Math.min(page * perPage, filteredProducts.length) }} з {{ filteredProducts.length }}
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const page = ref(1)
const perPage = ref(10)
const search = ref('')
const sort = ref({ column: 'title', direction: 'asc' })

// Define table columns
const columns = [
  {
    key: 'title',
    label: 'Назва',
    sortable: true
  },
  {
    key: 'description',
    label: 'Опис',
    sortable: true
  },
  {
    key: 'price',
    label: 'Ціна',
    sortable: true
  },
  {
    key: 'rating',
    label: 'Оцінка',
    sortable: true
  },
  {
    key: 'brand',
    label: 'Бренд',
    sortable: true
  },
  {
    key: 'category',
    label: 'Категорія',
    sortable: true
  },
  {
    key: 'thumbnail',
    label: 'Фото',
    sortable: false
  }
]

// Fetch products from API
const { data: productsData, pending, error } = await useFetch('https://dummyjson.com/products?limit=100')
const products = computed(() => productsData.value?.products || [])

// Get rating color based on value - using explicit hex colors
const getRatingColor = (rating) => {
  return rating >= 4.5 ? '#22c55e' : '#ef4444'  // Green if >= 4.5, Red if < 4.5
}

// Update sorting
const updateSort = (column) => {
  if (sort.value.column === column) {
    sort.value.direction = sort.value.direction === 'asc' ? 'desc' : 'asc'
  } else {
    sort.value.column = column
    sort.value.direction = 'asc'
  }
}

// Filter products based on search term
const filteredProducts = computed(() => {
  let result = [...products.value]

  // Apply search filter
  if (search.value) {
    const searchTerm = search.value.toLowerCase()
    result = result.filter(product =>
        product.title.toLowerCase().includes(searchTerm) ||
        product.description.toLowerCase().includes(searchTerm) ||
        product.brand.toLowerCase().includes(searchTerm) ||
        product.category.toLowerCase().includes(searchTerm)
    )
  }

  // Apply sorting (globally, not just on current page)
  if (sort.value.column) {
    result.sort((a, b) => {
      const aValue = a[sort.value.column]
      const bValue = b[sort.value.column]

      if (typeof aValue === 'string') {
        return sort.value.direction === 'asc'
            ? aValue.localeCompare(bValue)
            : bValue.localeCompare(aValue)
      } else {
        return sort.value.direction === 'asc'
            ? aValue - bValue
            : bValue - aValue
      }
    })
  }

  return result
})

// Paginated products
const paginatedProducts = computed(() => {
  const start = (page.value - 1) * perPage.value
  const end = start + perPage.value
  return filteredProducts.value.slice(start, end)
})

// Calculate total pages
const totalPages = computed(() => {
  return Math.ceil(filteredProducts.value.length / perPage.value)
})

// Calculate which page numbers to display
const displayedPages = computed(() => {
  const maxDisplayed = 5
  const halfDisplayed = Math.floor(maxDisplayed / 2)

  if (totalPages.value <= maxDisplayed) {
    return Array.from({ length: totalPages.value }, (_, i) => i + 1)
  }

  let start = Math.max(1, page.value - halfDisplayed)
  let end = start + maxDisplayed - 1

  if (end > totalPages.value) {
    end = totalPages.value
    start = Math.max(1, end - maxDisplayed + 1)
  }

  return Array.from({ length: end - start + 1 }, (_, i) => start + i)
})

// Reset to first page when search changes
watch(search, () => {
  page.value = 1
})

// Reset to first page when perPage changes
watch(perPage, () => {
  page.value = 1
})
</script>

<style>
:root {
  --color-purple-50: #faf5ff;
  --color-purple-100: #f3e8ff;
  --color-purple-200: #e9d5ff;
  --color-purple-300: #d8b4fe;
  --color-purple-400: #c084fc;
  --color-purple-500: #a855f7;
  --color-purple-600: #9333ea;
  --color-purple-700: #7e22ce;
  --color-purple-800: #6b21a8;
  --color-purple-900: #581c87;

  --color-gray-50: #f9fafb;
  --color-gray-100: #f3f4f6;
  --color-gray-200: #e5e7eb;
  --color-gray-300: #d1d5db;
  --color-gray-400: #9ca3af;
  --color-gray-500: #6b7280;
  --color-gray-600: #4b5563;
  --color-gray-700: #374151;
  --color-gray-800: #1f2937;
  --color-gray-900: #111827;

  --color-green-500: #22c55e;
  --color-red-500: #ef4444;

  --shadow-sm: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
  --shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  --shadow-md: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
  --shadow-lg: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);

  --radius-sm: 0.125rem;
  --radius: 0.25rem;
  --radius-md: 0.375rem;
  --radius-lg: 0.5rem;
  --radius-xl: 0.75rem;
  --radius-2xl: 1rem;

  --font-sans: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: var(--font-sans);
  color: var(--color-gray-800);
  background-color: var(--color-gray-100);
  line-height: 1.5;
}

.container {
  min-height: 100vh;
  padding: 2rem;
  background-color: var(--color-gray-100);
}

.content {
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  margin-bottom: 2rem;
  text-align: center;
}

.title {
  font-size: 2.25rem;
  font-weight: 700;
  color: var(--color-purple-800);
  margin-bottom: 0.5rem;
}

.subtitle {
  font-size: 1.125rem;
  color: var(--color-gray-600);
}

.card {
  background-color: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
  overflow: hidden;
  margin-bottom: 1.5rem;
}

.search-card {
  padding: 1rem;
}

.search-container {
  position: relative;
}

.search-icon {
  position: absolute;
  left: 1rem;
  top: 50%;
  transform: translateY(-50%);
  color: var(--color-gray-400);
}

.search-input {
  width: 100%;
  padding: 0.75rem 1rem 0.75rem 2.5rem;
  border: 1px solid var(--color-gray-300);
  border-radius: var(--radius-md);
  font-size: 1rem;
  transition: all 0.2s ease;
}

.search-input:focus {
  outline: none;
  border-color: var(--color-purple-500);
  box-shadow: 0 0 0 3px rgba(168, 85, 247, 0.1);
}

.loading-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem;
  background-color: white;
  border-radius: var(--radius-lg);
  box-shadow: var(--shadow-md);
}

.loading-spinner {
  width: 40px;
  height: 40px;
  border: 3px solid var(--color-purple-200);
  border-top-color: var(--color-purple-600);
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 1rem;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.table-card {
  overflow: hidden;
}

.table-container {
  overflow-x: auto;
}

.products-table {
  width: 100%;
  border-collapse: collapse;
}

.table-header {
  padding: 1rem;
  text-align: left;
  font-weight: 600;
  font-size: 0.875rem;
  color: var(--color-gray-700);
  background-color: var(--color-gray-50);
  border-bottom: 1px solid var(--color-gray-200);
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.sortable {
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.sortable:hover {
  background-color: var(--color-gray-100);
}

.sort-icon {
  display: inline-flex;
  align-items: center;
  color: var(--color-gray-500);
}

.sort-default {
  opacity: 0.3;
}

.table-row {
  transition: background-color 0.2s ease;
}

.table-row:hover {
  background-color: var(--color-purple-50);
}

.table-cell {
  padding: 1rem;
  border-bottom: 1px solid var(--color-gray-200);
}

.description-cell {
  max-width: 300px;
}

.truncated {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.price-cell {
  font-weight: 600;
}

.rating {
  display: inline-flex;
  align-items: center;
  padding: 0.25rem 0.5rem;
  border-radius: var(--radius);
  font-weight: 600;
}

.rating-high {
  background-color: rgba(34, 197, 94, 0.1);
  color: var(--color-green-500);
}

.rating-low {
  background-color: rgba(239, 68, 68, 0.1);
  color: var(--color-red-500);
}

.rating-star {
  margin-left: 0.25rem;
}

.category-tag {
  display: inline-block;
  padding: 0.25rem 0.5rem;
  background-color: var(--color-purple-100);
  color: var(--color-purple-800);
  border-radius: var(--radius);
  font-size: 0.75rem;
  font-weight: 500;
}

.image-cell {
  text-align: center;
}

.product-image {
  width: 100px;
  height: 100px;
  object-fit: cover;
  border-radius: var(--radius);
  box-shadow: var(--shadow-sm);
  transition: transform 0.2s ease;
}

.product-image:hover {
  transform: scale(1.05);
}

.empty-message {
  padding: 3rem;
  text-align: center;
}

.empty-content {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: var(--color-gray-500);
}

.empty-content svg {
  margin-bottom: 1rem;
}

.empty-hint {
  font-size: 0.875rem;
  margin-top: 0.5rem;
}

.table-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem;
  border-top: 1px solid var(--color-gray-200);
  background-color: var(--color-gray-50);
}

.per-page-selector {
  display: flex;
  align-items: center;
}

.per-page-label {
  margin-right: 0.5rem;
  font-size: 0.875rem;
  color: var(--color-gray-600);
}

.per-page-select {
  padding: 0.375rem 0.75rem;
  border: 1px solid var(--color-gray-300);
  border-radius: var(--radius);
  background-color: white;
  font-size: 0.875rem;
}

.pagination {
  display: flex;
  align-items: center;
}

.page-button {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 36px;
  height: 36px;
  margin: 0 0.125rem;
  border: 1px solid var(--color-gray-300);
  border-radius: var(--radius);
  background-color: white;
  color: var(--color-gray-700);
  cursor: pointer;
  transition: all 0.2s ease;
}

.page-button:hover:not(:disabled):not(.active) {
  background-color: var(--color-gray-100);
  border-color: var(--color-gray-400);
}

.page-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-button.active {
  background-color: var(--color-purple-600);
  border-color: var(--color-purple-600);
  color: white;
}

.page-numbers {
  display: flex;
  margin: 0 0.25rem;
}

.status-info {
  font-size: 0.875rem;
  color: var(--color-gray-600);
}

@media (max-width: 768px) {
  .container {
    padding: 1rem;
  }

  .table-footer {
    flex-direction: column;
    gap: 1rem;
  }

  .pagination {
    order: -1;
  }
}
</style>