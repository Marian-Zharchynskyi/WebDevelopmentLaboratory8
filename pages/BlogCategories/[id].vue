<template>
  <div class="container mx-auto p-4">

    <div class="max-w-md mx-auto">
      <UCard class="mb-8">
        <template #header>
          <h2 class="text-2xl font-bold mb-2">Деталі категорії</h2>
          <p class="text-gray-600">ID категорії: {{ categoryId }}</p>
        </template>
        <ul>
          <li class="border-b-2 py-2">
            <span class="font-semibold">Назва:</span> {{ category?.title }}
          </li>
          <li class="border-b-2 py-2">
            <span class="font-semibold">Посилання:</span> {{ category?.slug }}
          </li>
          <li class="border-b-2 py-2">
            <span class="font-semibold">Опис:</span> {{ category?.description || 'Відсутній' }}
          </li>
          <li class="border-b-2 py-2">
            <span class="font-semibold">Дата створення:</span> {{ category?.created_at || 'Відсутня' }}
          </li>
          <li class="border-b-2 py-2">
            <span class="font-semibold">Дата оновлення:</span> {{ category?.updated_at || 'Відсутня' }}
          </li>
          <li class="py-2">
            <span class="font-semibold">Дата видалення:</span> {{ category?.deleted_at || 'Відсутня' }}
          </li>
        </ul>
        <template #footer>
          <p class="text-gray-600">ID батьківської категорії: {{ category?.parent_id || 'Відсутній' }}</p>
        </template>
      </UCard>
    </div>
    <div class="text-center">
      <nuxt-link class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded inline-block mb-4" to="/BlogCategories">
        Назад до категорій
      </nuxt-link>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';
import axios from 'axios';

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
  parent_id: number;
}

const route = useRoute();
const categoryId = ref<number | null>(null);
const category = ref<Category | null>(null);

const getCategory = async (id: number) => {
  try {
    const response = await axios.get<Category>(`http://127.0.0.1:8000/api/blog/categories/${id}`);
    category.value = response.data;
  } catch (error) {
    console.error('Error fetching category:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    categoryId.value = id;
    getCategory(id);
  } else {
    console.error('Invalid Category ID');
  }
});
</script>
