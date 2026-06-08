//CatalogPage.vue
<template>
  <div class="catalog-page">
    <div class="container">
      <!-- Если идёт поиск, показываем результаты поиска -->
      <div v-if="searchQuery" class="search-results-section">
        <div class="search-header">
          <h1>Результаты поиска</h1>
          <div class="search-query-display">
            <span class="search-label">Запрос:</span>
            <span class="search-value">"{{ searchQuery }}"</span>
            <router-link to="/catalog" class="clear-search">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="16"
                height="16"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
              >
                <line x1="18" y1="6" x2="6" y2="18"></line>
                <line x1="6" y1="6" x2="18" y2="18"></line>
              </svg>
              Очистить
            </router-link>
          </div>
        </div>

        <div v-if="loading" class="loading-container">
          <div class="loading-spinner"></div>
          <p>Поиск товаров...</p>
        </div>

        <div
          v-else-if="searchResults.length === 0"
          class="no-results-container"
        >
          <div class="no-results-icon">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              width="64"
              height="64"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="1.5"
            >
              <circle cx="11" cy="11" r="8"></circle>
              <path d="m21 21-4.35-4.35"></path>
              <line x1="11" y1="8" x2="11" y2="14"></line>
              <line x1="11" y1="16" x2="11.01" y2="16"></line>
            </svg>
          </div>
          <h2>Ничего не найдено</h2>
          <p>
            По запросу <strong>"{{ searchQuery }}"</strong> товары не найдены
          </p>
          <router-link to="/catalog" class="back-to-catalog">
            Вернуться в каталог
          </router-link>
        </div>

        <div v-else>
          <div class="search-results-info">
            Найдено товаров: <strong>{{ searchResults.length }}</strong>
          </div>
          <div class="products-grid">
            <ProductCard
              v-for="product in searchResults"
              :key="product.id"
              :product="product"
            />
          </div>
        </div>
      </div>

      <!-- Обычное отображение категорий -->
      <div v-else>
        <h1>Каталог товаров</h1>

        <div v-if="loading" class="loading-container">
          <div class="loading-spinner"></div>
          <p>Загрузка каталога...</p>
        </div>

        <div v-else-if="error" class="error-message">
          {{ error }}
        </div>

        <div v-else-if="categories.length" class="categories-section">
          <div
            v-for="category in categories"
            :key="category.id"
            class="category-block"
          >
            <div class="category-header">
              <h2>{{ category.name }}</h2>
              <router-link
                :to="`/catalog/${category.slug}`"
                class="view-category-link"
              >
                Все товары ({{ category.products_count }}) →
              </router-link>
            </div>

            <div v-if="category.children.length" class="subcategories-grid">
              <div
                v-for="subcat in category.children"
                :key="subcat.id"
                class="subcategory-card"
              >
                <router-link :to="`/catalog/${subcat.slug}`">
                  <div class="subcat-image" v-if="subcat.image">
                    <img :src="subcat.image" :alt="subcat.name" />
                  </div>
                  <div class="subcat-content">
                    <h3>{{ subcat.name }}</h3>
                    <span class="products-count"
                      >{{ subcat.products_count }} товаров</span
                    >
                  </div>
                </router-link>
              </div>
            </div>

            <div v-else class="no-subcategories">
              <router-link
                :to="`/catalog/${category.slug}`"
                class="category-main-link"
              >
                <div class="category-main-image" v-if="category.image">
                  <img :src="category.image" :alt="category.name" />
                </div>
                <div v-else class="placeholder-image">
                  <span>{{ category.name[0] }}</span>
                </div>
                <p>Перейти к товарам →</p>
              </router-link>
            </div>
          </div>
        </div>

        <div v-else class="empty-catalog">
          <p>Каталог пуст</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue";
import { useRoute } from "vue-router";
import { categoriesAPI, productsAPI } from "@/api";
import type { Category, Product } from "@/types";
import ProductCard from "@/components/ProductCard.vue";

const route = useRoute();
const categories = ref<Category[]>([]);
const searchResults = ref<Product[]>([]);
const loading = ref(true);
const error = ref<string | null>(null);

const searchQuery = computed(() => (route.query.search as string) || "");

// Загрузка категорий
const loadCategories = async () => {
  try {
    const response = await categoriesAPI.getAll();
    const payload =
      "results" in response.data ? response.data.results : response.data;
    categories.value = payload;
  } catch (err) {
    error.value = "Ошибка загрузки каталога";
    console.error(err);
  }
};

// Загрузка результатов поиска
const loadSearchResults = async (query: string) => {
  try {
    loading.value = true;
    const response = await productsAPI.search(query, { page_size: 100 });
    const payload =
      "results" in response.data ? response.data.results : response.data;
    searchResults.value = Array.isArray(payload) ? payload : [];
  } catch (err) {
    console.error("Ошибка поиска товаров:", err);
    searchResults.value = [];
  } finally {
    loading.value = false;
  }
};

onMounted(async () => {
  try {
    loading.value = true;
    await loadCategories();

    // Если есть поисковый запрос, загружаем результаты
    if (searchQuery.value) {
      await loadSearchResults(searchQuery.value);
    }
  } catch (err) {
    error.value = "Ошибка загрузки данных";
    console.error("Failed to load data:", err);
  } finally {
    loading.value = false;
  }
});

// Реактивное обновление при изменении поиска
watch(
  () => route.query.search,
  async (newSearch) => {
    if (newSearch) {
      await loadSearchResults(newSearch as string);
    } else {
      searchResults.value = [];
    }
    // Скролл вверх при новом поиске
    window.scrollTo({ top: 0, behavior: "smooth" });
  },
);
</script>

<!-- Styles in main.css -->
