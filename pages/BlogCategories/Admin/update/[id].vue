<template>
  <div class="w-1/3 m-auto">

    <div class="p-5 border-2 rounded">
      <h1 class="text-center mb-2">Редагування категорії {{category?.title}}</h1>
      <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        <UFormGroup label="Назва" name="title">
          <UInput v-model="state.title" />
        </UFormGroup>

        <UFormGroup label="Посилання" name="slug">
          <UInput v-model="state.slug" />
        </UFormGroup>

        <UFormGroup label="Опис" name="description">
          <UInput v-model="state.description" />
        </UFormGroup>

        <UFormGroup label="Батьківська категорія" name="parent_id">
          <USelect v-model="state.parent_id" :options="categoryOptions" />
        </UFormGroup>
      </UForm>
    </div>
    <div class="p-5 flex justify-between items-center">
      <nuxt-link class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded inline-block mb-4" to="/BlogCategories">
        Назад до категорій
      </nuxt-link>
      <UButton @click="onSubmit" type="submit">
        Зберегти
      </UButton>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { computed, reactive, ref } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

interface Category {
  id: number;
  title: string;
  slug: string;
  description: string;
  created_at: string;
  updated_at: string;
  deleted_at: string;
  parent_id: number;
  parent_category: Category;
}

const route = useRoute();
const categoryId = ref<number | null>(null);
const category = ref<Category | null>(null);

const getCategory = async (id: number) => {
  try {
    const response = await $fetch<Category>(`http://127.0.0.1:8000/api/blog/categories/${id}`);
    category.value = response;
    state.title = category.value.title;
    state.slug = category.value.slug;
    state.description = category.value.description;
    state.parent_id = category.value.parent_id.toString();
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

const categories = ref<Category[]>([]);
const getCategories = async () => {
  try {
    const response = await $fetch<Category[]>(`http://127.0.0.1:8000/api/blog/categories`);
    categories.value = response;
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

// Computed property for category options
const categoryOptions = computed(() => {
  return categories.value.map(category => ({
    value: category.id.toString(), // Convert id to string
    label: category.title
  }));
});

// Define Zod schema for validation
const schema = z.object({
  title: z.string().min(5, 'Назва повинна містити щонайменше 5 символів').max(200, 'Назва повинна бути менше 200 символів'),
  slug: z.string().max(200, 'Силка повинна бути менше 200 символів').optional(),
  description: z.string().min(3, 'Опис повинен містити щонайменше 3 символи').max(500, 'Опис повинен бути менше 500 символів').optional(),
  parent_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Батьківська категорія повинна бути цілим числом і існувати в списку категорій')
});

type Schema = z.infer<typeof schema>;

const state = reactive<Partial<Schema>>({
  title: category.value?.title,
  slug: category.value?.slug,
  description: category.value?.description,
  parent_id: category.value?.parent_id.toString()
});

async function onSubmit(event: Event) {
  try {
    const result = schema.parse(state);
    console.log(result);

    // Convert parent_id to number before sending to server
    const dataToSend = {
      ...result,
      parent_id: Number(result.parent_id)
    };

    // Send the data to the server
    const response = await axios.put(`http://127.0.0.1:8000/api/blog/categories/${categoryId.value}`, dataToSend);
    alert("Категорію успішно оновлено!");
    window.location.href = '/BlogCategories';
  } catch (e) {
    if (e instanceof z.ZodError) {
      // Handle validation errors
      console.error(e.errors);
    } else {
      // Handle other errors
      console.error('Помилка при відправці форми:', e);
    }
  }
}
</script>
