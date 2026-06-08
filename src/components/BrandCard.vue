<!-- src/components/BrandCard.vue -->
<template>
  <div class="brand-card" @click="navigateToBrand">
    <div class="brand-image">
      <img
        v-if="brand.image"
        :src="brand.image"
        :alt="brand.name"
        class="brand-logo"
        loading="lazy"
        decoding="async"
      />
      <div v-else class="brand-placeholder">
        {{ brand.name[0] }}
      </div>
    </div>
    <div class="brand-info">
      <h3>{{ brand.name }}</h3>
      <p class="products-count">{{ brand.products_count }} товаров</p>
      <p v-if="brand.description" class="description">
        {{ brand.description }}
      </p>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";
import type { Brand } from "@/types";

const props = defineProps<{
  brand: Brand;
}>();

const router = useRouter();

const navigateToBrand = () => {
  router.push(`/brands/${props.brand.slug}`);
};
</script>

<!-- Styles in main.css -->
