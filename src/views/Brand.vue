<!-- ========================================
     src/views/Brand.vue
     ======================================== -->
<template>
  <div class="brand-page">
    <div class="container">
      <!-- Хлебные крошки -->
      <nav class="breadcrumb">
        <router-link to="/">Главная</router-link>
        <span class="separator">›</span>
        <router-link to="/brands">Бренды</router-link>
        <span class="separator">›</span>
        <span class="current">{{ brand?.name || "Загрузка..." }}</span>
      </nav>

      <!-- Состояние загрузки -->
      <div v-if="loading" class="loading-container">
        <div class="spinner"></div>
        <p>Загрузка информации о бренде...</p>
      </div>
      <!-- Контент бренда -->
      <div v-else-if="brand" class="brand-content">
        <!-- Шапка бренда -->
        <div class="brand-header">
          <div class="brand-logo" v-if="brand.image">
            <img :src="getImageUrl(brand.image)" :alt="brand.name" />
          </div>
          <div class="brand-info">
            <h1>{{ brand.name }}</h1>
            <p
              v-if="brand.description"
              class="brand-description"
              v-html="formatContent(brand.description)"
            ></p>
          </div>
        </div>

        <div class="brand-main">
          <aside class="brand-filters">
            <div class="panel">
              <h3>Фильтры</h3>

              <!-- Категории внутри бренда -->
              <div class="filter-group" v-if="brandCategories.length">
                <label>Категории</label>
                <div class="categories-list">
                  <label
                    v-for="c in brandCategories"
                    :key="c.slug"
                    class="category-item"
                  >
                    <input
                      type="checkbox"
                      :value="c.slug"
                      v-model="filters.categories"
                      @change="applyFilters"
                    />
                    <span>{{ c.name }}</span>
                  </label>
                </div>
              </div>

              <!-- Теги -->
              <div class="filter-group" v-if="brandTags.length">
                <label>Теги</label>
                <div class="tags-list">
                  <label v-for="t in brandTags" :key="t.slug" class="tag-item">
                    <input
                      type="checkbox"
                      :value="t.slug"
                      v-model="filters.tags"
                      @change="applyFilters"
                    />
                    <span>{{ t.name }}</span>
                  </label>
                </div>
              </div>

              <!-- Цена -->
              <div class="filter-group">
                <label>Цена</label>
                <div class="price-range">
                  <input
                    type="number"
                    v-model.number="filters.priceMin"
                    placeholder="От"
                    @change="applyFilters"
                  />
                  <span>—</span>
                  <input
                    type="number"
                    v-model.number="filters.priceMax"
                    placeholder="До"
                    @change="applyFilters"
                  />
                </div>
              </div>

              <div class="filter-actions">
                <button class="btn muted" @click="resetFilters">
                  Сбросить
                </button>
              </div>
            </div>
          </aside>

          <!-- Товары бренда -->
          <section v-if="products.length" class="brand-products">
            <div class="section-header">
              <h2>Товары бренда</h2>
              <span class="products-count"
                >{{ products.length }} {{ pluralize(products.length) }}</span
              >
            </div>

            <ProductGrid :products="products" />

            <!-- Пагинация -->
            <Pagination
              v-if="pagination.totalPages > 1"
              :current-page="pagination.page"
              :total-pages="pagination.totalPages"
              @change="changePage"
            />
          </section>
        </div>
      </div>
      <!-- Состояние ошибки -->
      <div v-else="error" class="error-state">
        <svg
          xmlns="http://www.w3.org/2000/svg"
          width="64"
          height="64"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2"
        >
          <circle cx="12" cy="12" r="10"></circle>
          <line x1="12" y1="8" x2="12" y2="12"></line>
          <line x1="12" y1="16" x2="12.01" y2="16"></line>
        </svg>
        <p class="error-message">{{ error }}</p>
        <router-link to="/brands" class="btn-back">
          <svg
            xmlns="http://www.w3.org/2000/svg"
            width="20"
            height="20"
            viewBox="0 0 24 24"
            fill="none"
            stroke="currentColor"
            stroke-width="2"
          >
            <line x1="19" y1="12" x2="5" y2="12"></line>
            <polyline points="12 19 5 12 12 5"></polyline>
          </svg>
          Вернуться к брендам
        </router-link>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, watch } from "vue";
import { useRoute } from "vue-router";
import { brandsAPI, getImageUrl } from "@/api";
import type { Brand, Product } from "@/types";
import ProductGrid from "@/components/ProductGrid.vue";
import Pagination from "@/components/Pagination.vue";

const route = useRoute();

const brand = ref<Brand | null>(null);
const products = ref<Product[]>([]);
const loading = ref(true);
const error = ref<string | null>(null);

const brandCategories = ref<any[]>([]);
const brandTags = ref<any[]>([]);

const filters = reactive({
  categories: [] as string[],
  tags: [] as string[],
  priceMin: null as number | null,
  priceMax: null as number | null,
  ordering: "-created_at",
});

const pagination = reactive({
  page: 1,
  pageSize: 12,
  totalPages: 1,
  total: 0,
});

// const formatPrice = (price: number): string => {
//   return new Intl.NumberFormat("ru-RU").format(price);
// };

const formatContent = (content: string): string => {
  return content ? content.replace(/\n/g, "<br>") : "";
};

const pluralize = (count: number): string => {
  const cases = [2, 0, 1, 1, 1, 2];
  const titles = ["товар", "товара", "товаров"];
  return titles[
    count % 100 > 4 && count % 100 < 20
      ? 2
      : cases[count % 10 < 5 ? count % 10 : 5]
  ];
};

const loadBrand = async (): Promise<void> => {
  const slug = route.params.slug as string;
  if (!slug) {
    error.value = "Бренд не найден";
    loading.value = false;
    return;
  }

  try {
    loading.value = true;
    error.value = null;

    // Загрузка информации о бренде
    const brandResponse = await brandsAPI.getBySlug(slug);
    brand.value = brandResponse.data;

    // Загрузка категорий и тегов для бренда
    await Promise.all([loadBrandCategories(slug), loadBrandTags(slug)]);
    // Загрузка товаров бренда
    await loadProducts(slug);
  } catch (err: any) {
    console.error("Ошибка загрузки бренда:", err);
    error.value =
      err.response?.status === 404
        ? "Бренд не найден"
        : "Не удалось загрузить информацию о бренде";
  } finally {
    loading.value = false;
  }
};

const buildBrandParams = () => {
  const params: Record<string, any> = {
    page: pagination.page,
    page_size: pagination.pageSize,
    ordering: filters.ordering,
  };
  if (filters.categories.length) params.category = filters.categories.join(",");
  if (filters.tags.length) params.tag = filters.tags.join(",");
  if (filters.priceMin != null) params.price_min = filters.priceMin;
  if (filters.priceMax != null) params.price_max = filters.priceMax;
  return params;
};

const loadProducts = async (slug: string): Promise<void> => {
  try {
    const params = buildBrandParams();
    const response = await brandsAPI.getProducts(slug, params);

    const data = response.data;
    products.value = data.results || [];
    pagination.total = Number(data.count ?? 0);
    pagination.totalPages = Math.max(
      1,
      Math.ceil(pagination.total / pagination.pageSize)
    );
  } catch (err) {
    console.error("Ошибка загрузки товаров:", err);
    products.value = [];
  }
};

const loadBrandCategories = async (slug: string) => {
  try {
    const res = await brandsAPI.getCategories(slug);
    brandCategories.value = res.data || [];
  } catch (e) {
    console.error("Ошибка загрузки категорий бренда", e);
  }
};

const loadBrandTags = async (slug: string) => {
  try {
    const res = await brandsAPI.getTags(slug);
    const data = res.data || [];

    // Flatten grouped tags into a single array
    if (Array.isArray(data) && data.length > 0 && (data[0] as any).group_name) {
      // Data is in grouped format from ProductTagGroup
      const flatTags: any[] = [];
      const seenIds = new Set<number>();

      for (const group of data as any[]) {
        if (group.tags && Array.isArray(group.tags)) {
          for (const tag of group.tags) {
            if (!seenIds.has(tag.id)) {
              flatTags.push(tag);
              seenIds.add(tag.id);
            }
          }
        }
      }
      brandTags.value = flatTags;
    } else {
      // Data is already flat (shouldn't happen with current API but for safety)
      brandTags.value = data;
    }
  } catch (e) {
    console.error("Ошибка загрузки тегов бренда", e);
  }
};

const applyFilters = async () => {
  pagination.page = 1;
  const slug = route.params.slug as string;
  await loadProducts(slug);
};

const resetFilters = async () => {
  filters.categories = [];
  filters.tags = [];
  filters.priceMin = null;
  filters.priceMax = null;
  filters.ordering = "-created_at";
  await applyFilters();
};

const changePage = async (page: number): Promise<void> => {
  if (page < 1 || page > pagination.totalPages) return;
  pagination.page = page;
  await applyFilters();
  window.scrollTo({ top: 0, behavior: "smooth" });
};

onMounted(() => {
  loadBrand();
});

watch(
  () => route.params.slug,
  () => {
    pagination.page = 1;
    loadBrand();
  }
);
</script>

<!-- Styles in main.css -->
