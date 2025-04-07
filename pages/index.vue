<template>
  <div class="page-wrapper">
    <div class="content-box">
      <h1>👋 Вітаємо у списку продуктів</h1>
      <p>
        Перейдіть на
        <NuxtLink to="/products" class="product-link">Список продуктів</NuxtLink>, щоб побачити всі товари.
      </p>
    </div>
  </div>
</template>

<style scoped>
.page-wrapper {
  min-height: 100vh;
  background: linear-gradient(to bottom right, #f3f4f6, #ffffff);
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
}

.content-box {
  background-color: white;
  border-radius: 16px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.08);
  padding: 32px;
  max-width: 600px;
  width: 100%;
}

h1 {
  font-size: 28px;
  color: #1f2937;
  margin-bottom: 16px;
}

p {
  font-size: 16px;
  color: #4b5563;
}

.product-link {
  color: #2563eb;
  text-decoration: underline;
  transition: color 0.2s ease-in-out;
}

.product-link:hover {
  color: #1d4ed8;
}
</style>
