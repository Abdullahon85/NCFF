<template>
  <header class="header">
    <div class="container">
      <nav class="navbar">
        <router-link to="/" class="logo" @click="mobileMenuOpen = false">
          <span></span>
        </router-link>

        <!-- Search Bar -->
        <div class="search-container">
          <form
            @submit.prevent="navigateToSearchResults"
            class="search-wrapper"
          >
            <input
              type="text"
              v-model="searchQuery"
              @keyup.enter="navigateToSearchResults"
              placeholder="Поиск по названию или артикулу..."
              class="search-input"
            />
            <button
              type="submit"
              class="search-btn"
              :disabled="searchQuery.trim().length < 1"
            >
              <svg
                xmlns="http://www.w3.org/2000/svg"
                width="20"
                height="20"
                viewBox="0 0 24 24"
                fill="none"
                stroke="currentColor"
                stroke-width="2"
                stroke-linecap="round"
                stroke-linejoin="round"
              >
                <circle cx="11" cy="11" r="8"></circle>
                <path d="m21 21-4.35-4.35"></path>
              </svg>
            </button>
          </form>
        </div>

        <!-- Favorites, Compare & Cart icons -->
        <div class="header-icons">
          <router-link
            to="/favorites"
            class="header-icon-btn"
            title="Избранное"
            @click="mobileMenuOpen = false"
          >
            <svg
              width="22"
              height="22"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path
                d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"
              />
            </svg>
            <span v-if="favCount > 0" class="header-icon-badge">{{
              favCount
            }}</span>
          </router-link>
          <router-link
            to="/compare"
            class="header-icon-btn"
            title="Сравнение"
            @click="mobileMenuOpen = false"
          >
            <svg
              width="22"
              height="22"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <line x1="18" y1="20" x2="18" y2="10" />
              <line x1="12" y1="20" x2="12" y2="4" />
              <line x1="6" y1="20" x2="6" y2="14" />
            </svg>
            <span v-if="compareCount > 0" class="header-icon-badge">{{
              compareCount
            }}</span>
          </router-link>
          <router-link
            to="/cart"
            class="header-icon-btn"
            title="Корзина"
            @click="mobileMenuOpen = false"
          >
            <svg
              width="22"
              height="22"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <circle cx="9" cy="21" r="1" />
              <circle cx="20" cy="21" r="1" />
              <path
                d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"
              />
            </svg>
            <span v-if="cartCount > 0" class="header-icon-badge">{{
              cartCount
            }}</span>
          </router-link>
        </div>

        <button
          class="mobile-menu-toggle"
          @click="mobileMenuOpen = !mobileMenuOpen"
          :aria-expanded="mobileMenuOpen"
          :aria-label="mobileMenuOpen ? 'Закрыть меню' : 'Открыть меню'"
        >
          <span></span>
          <span></span>
          <span></span>
        </button>

        <ul class="nav-menu" :class="{ active: mobileMenuOpen }">
          <!-- Search inside mobile menu -->
          <li class="mobile-search-item">
            <form
              @submit.prevent="navigateToSearchResults"
              class="mobile-search-wrapper"
            >
              <input
                type="text"
                v-model="searchQuery"
                @keyup.enter="navigateToSearchResults"
                placeholder="Поиск по названию или артикулу..."
                class="mobile-search-input"
              />
              <button
                type="submit"
                class="mobile-search-btn"
                :disabled="searchQuery.trim().length < 1"
              >
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  width="18"
                  height="18"
                  viewBox="0 0 24 24"
                  fill="none"
                  stroke="currentColor"
                  stroke-width="2"
                  stroke-linecap="round"
                  stroke-linejoin="round"
                >
                  <circle cx="11" cy="11" r="8"></circle>
                  <path d="m21 21-4.35-4.35"></path>
                </svg>
              </button>
            </form>
          </li>

          <li>
            <router-link to="/" @click="mobileMenuOpen = false"
              >Главная</router-link
            >
          </li>

          <!-- CATALOG -->
          <li
            class="catalog-menu"
            @mouseenter="!isMobile && openMenu()"
            @mouseleave="!isMobile && closeMenu()"
          >
            <router-link to="/catalog" @click="handleCatalogClick"
              >Каталог</router-link
            >
          </li>

          <li>
            <router-link to="/about" @click="mobileMenuOpen = false"
              >О нас</router-link
            >
          </li>
          <li>
            <router-link to="/contact" @click="mobileMenuOpen = false"
              >Контакты</router-link
            >
          </li>
        </ul>

        <!-- MEGA MENU -->
        <div
          v-if="open && !isMobile"
          class="mega-menu"
          @mouseenter="openMenu()"
          @mouseleave="closeMenu()"
        >
          <div class="mega-menu-inner">
            <router-link
              v-for="cat in categories"
              :key="cat.id"
              :to="`/catalog/${cat.slug}`"
              class="mega-menu-item"
              @click="open = false; mobileMenuOpen = false"
            >
              {{ cat.name }}
            </router-link>
          </div>
        </div>

      </nav>
    </div>
  </header>
</template>

<script setup lang="ts">
import { ref, computed, onMounted, onBeforeUnmount, watch } from "vue";
import { useRoute, useRouter } from "vue-router";
import { categoriesAPI } from "@/api";
import type { Category } from "@/types";
import { useFavoritesStore } from "@/stores/useFavoritesStore";
import { useCompareStore } from "@/stores/useCompareStore";
import { useCartStore } from "@/stores/useCartStore";

const route = useRoute();
const router = useRouter();
const mobileMenuOpen = ref(false);
const open = ref(false);
const categories = ref<Category[]>([]);

const favStore = useFavoritesStore();
const compareStore = useCompareStore();
const cartStore = useCartStore();
const favCount = computed(() => favStore.items.length);
const compareCount = computed(() => compareStore.items.length);
const cartCount = computed(() => cartStore.totalQty);

const searchQuery = ref("");

const isMobile = ref(window.innerWidth <= 768);
const onResize = () => { isMobile.value = window.innerWidth <= 768; };
window.addEventListener("resize", onResize);
onBeforeUnmount(() => window.removeEventListener("resize", onResize));

// Задержка чтобы мышь успевала перейти с li на мега-меню
let closeTimer: ReturnType<typeof setTimeout> | null = null;

const openMenu = () => {
  if (closeTimer) clearTimeout(closeTimer);
  open.value = true;
};

const closeMenu = () => {
  closeTimer = setTimeout(() => {
    open.value = false;
  }, 100);
};

const handleCatalogClick = () => {
  mobileMenuOpen.value = false;
  open.value = false;
};

const navigateToSearchResults = () => {
  const query = searchQuery.value.trim();
  if (query.length >= 2) {
    router.push({
      path: "/catalog",
      query: { search: query },
    });
    searchQuery.value = "";
  }
};

watch(
  () => route.path,
  () => {
    mobileMenuOpen.value = false;
    open.value = false;
    searchQuery.value = "";
  },
);

watch(mobileMenuOpen, (isOpen) => {
  document.body.style.overflow = isOpen ? "hidden" : "";
});

onMounted(async () => {
  try {
    const res = await categoriesAPI.getAll(100);
    categories.value = res.data.results;
  } catch (e) {
    console.error("Ошибка загрузки каталога:", e);
  }
});
</script>

<!-- Styles in main.css -->