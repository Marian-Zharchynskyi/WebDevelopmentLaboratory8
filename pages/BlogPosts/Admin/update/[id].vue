<template>
  <div class="w-full max-w-lg mx-auto mt-10">
    <!-- Back button -->
    <div class="mb-6">

    </div>

    <!-- Form container -->
    <div class="bg-white shadow-md rounded-lg px-8 py-6">
      <!-- Post editing title -->
      <h1 class="text-2xl font-bold text-center mb-6">Редагування поста "{{ post?.title }}"</h1>

      <!-- Form -->
      <UForm :schema="schema" :state="state" class="space-y-4" @submit="onSubmit">
        <!-- Title field -->
        <UFormGroup label="Title" name="title">
          <UInput v-model="state.title" />
        </UFormGroup>

        <!-- Slug field -->
        <UFormGroup label="Slug" name="slug">
          <UInput v-model="state.slug" />
        </UFormGroup>

        <!-- Content field -->
        <UFormGroup label="Content" name="content">
          <UTextarea v-model="state.content_raw" />
        </UFormGroup>

        <!-- Excerpt field -->
        <UFormGroup label="Excerpt" name="excerpt">
          <UInput v-model="state.excerpt" />
        </UFormGroup>

        <!-- Category field -->
        <UFormGroup label="Category" name="category_id">
          <USelect v-model="state.category_id" :options="categoryOptions" />
        </UFormGroup>

        <!-- Is published checkbox -->
        <UFormGroup class="flex items-center space-x-2" label="Is published" name="is_published">
          <UCheckbox v-model="state.is_published" />
        </UFormGroup>

        <!-- Submit button -->
        <div class="flex justify-between">
          <UButton class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" type="submit">
            Save
          </UButton>
          <nuxt-link class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded inline-block" to="/BlogPosts">
            Back
          </nuxt-link>
        </div>
      </UForm>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed, onMounted } from 'vue';
import axios from 'axios';
import { useRoute } from 'vue-router';

interface Category {
  id: number;
  title: string;
}

interface Post {
  id: number;
  title: string;
  slug: string;
  content_raw: string;
  excerpt: string;
  category_id: number;
  is_published: boolean;
}

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

const route = useRoute();
const postId = ref<number | null>(null);
const post = ref<Post | null>(null);

const getPost = async (id: number) => {
  try {
    const response = await $fetch<Post>(`http://127.0.0.1:8000/api/blog/posts/${id}`);
    post.value = response;
    state.category_id = post.value.category_id.toString();
    state.title = post.value.title;
    state.slug = post.value.slug;
    state.is_published = post.value.is_published;
    state.excerpt = post.value.excerpt;
    state.content_raw = post.value.content_raw;
  } catch (error) {
    console.error('Error fetching post:', error);
  }
};

onMounted(() => {
  const id = parseInt(route.params.id as string, 10);
  if (!isNaN(id)) {
    postId.value = id;
    getPost(id);
  } else {
    console.error('Invalid post ID');
  }
});

const posts = ref<Post[]>([]);
const getPosts = async () => {
  try {
    const response = await $fetch<Post[]>(`http://127.0.0.1:8000/api/blog/posts`);
    posts.value = response;
  } catch (error) {
    console.error('Error fetching posts:', error);
  }
};
getPosts();

// Computed property for category options
const categoryOptions = computed(() => {
  return categories.value.map(category => ({
    value: category.id.toString(),
    label: category.title
  }));
});

// Define Zod schema for validation
const schema = z.object({
  title: z.string().min(5, 'Заголовок має містити щонайменше 5 символів').max(200, 'Заголовок має бути менше 200 символів')
      .refine(value => {
        return !posts.value.some(post => post.title === value && post.id !== postId.value)
      }, 'Заголовок повинен бути унікальним'),
  slug: z.string().max(200, 'Slug має бути менше 200 символів').
  refine(value => {
    return !posts.value.some(post => post.slug === value && post.id !== postId.value)
  }, 'Slug повинен бути унікальним').optional(),
  content_raw: z.string().min(5, 'Зміст має містити щонайменше 5 символів').max(10000, 'Зміст має бути менше 10000 символів'),
  category_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Категорія має бути валідним цілим числом і існувати у списку категорій'),
  is_published: z.boolean().optional(),
  excerpt: z.string().optional()
});

type Schema = z.infer<typeof schema>;

const state = reactive<Partial<Schema>>({
  title: undefined,
  slug: undefined,
  content_raw: undefined,
  category_id: undefined,
  is_published: undefined,
  excerpt: undefined
});

async function onSubmit (event: Event) {
  try {
    const result = schema.parse(state);
    console.log(result);

    // Convert category_id to number before sending to server
    const dataToSend = {
      ...result,
      category_id: Number(result.category_id)
    };

    // Send the data to the server
    const response = await axios.put(`http://127.0.0.1:8000/api/blog/posts/${postId.value}`, dataToSend);
    alert("Пост успішно оновлено!");
    window.location.href = '/BlogPosts';
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

