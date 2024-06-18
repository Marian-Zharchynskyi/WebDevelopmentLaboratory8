<template>
  <div class="container mx-auto p-4">

    <h1 class="text-3xl font-bold mb-4">Список категорій</h1>
    <UTable :columns="columns" :rows="paginated_rows" :total="total" class="shadow-lg rounded-lg overflow-hidden">
      <template #actions-data="{ row }">
        <UDropdown :items="items(row)" class="relative">
          <UButton color="gray" variant="ghost" icon="i-heroicons-ellipsis-horizontal-20-solid" class="text-gray-600" />
        </UDropdown>
      </template>
      <template #parent_title-data="{ row }">
        <span>{{ row.parent_title }}</span>
      </template>
    </UTable>
    <div class="p-5">
      <nuxt-link class="mt-20 bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded" to="/BlogCategories/Admin/create">Додати категорію</nuxt-link>
    </div>
    <div class="mt-4">
      <UPagination v-model="page" :page-count="page_count" :total="total" class="flex justify-center" />
    </div>
  </div>
</template>

<script setup lang="ts">
import axios from "axios";
import { ref, computed } from 'vue';

const columns = [
  { key: 'id', label: '#' },
  { key: 'title', label: 'Назва' },
  { key: 'slug', label: 'Посилання' },
  { key: 'description', label: 'Опис' },
  { key: 'created_at', label: 'Дата створення' },
  { key: 'updated_at', label: 'Дата оновлення' },
  { key: 'deleted_at', label: 'Дата видалення' },
  { key: 'parent_title', label: 'Батьківська категорія' },
  { key: 'actions', label: 'Дії' }
];

const page = ref(1);
const page_count = 5;

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
  parent_title: string;
}

const categories = ref<Category[]>([]);

const getCategories = async () => {
  try {
    const response = await axios.get<Category[]>(`http://127.0.0.1:8000/api/blog/categories`);
    categories.value = response.data;
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

const rows = computed(() => {
  return categories.value.map((category) => ({
    id: category.id,
    title: category.title || 'none',
    description: category.description || 'none',
    slug: category.slug || 'none',
    parent_title: category.parent_title || 'none',
    created_at: category.created_at || 'none',
    updated_at: category.updated_at || 'none',
    deleted_at: category.deleted_at || 'none'
  }));
});

const total = computed(() => rows.value.length);
const paginated_rows = computed(() => {
  return rows.value.slice((page.value - 1) * page_count, page.value * page_count);
});

const items = (category: Category) => [
  [{
    label: 'Редагувати',
    icon: 'i-heroicons-pencil-square-20-solid',
    click: () => window.location.href = `/BlogCategories/Admin/update/${category?.id}`
  }, {
    label: 'Деталі',
    icon: 'i-heroicons-information-circle-20-solid',
    click: () => window.location.href = `/BlogCategories/${category?.id}`
  }]
];
</script>
