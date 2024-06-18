<template>
  <div class="container mx-auto px-4 py-8">
    <div class="flex justify-between items-center mb-8">
      <h1 class="text-3xl font-bold">Create New Post</h1>
    </div>
    <div class="bg-white shadow-md rounded-lg p-8">
      <UForm :schema="schema" :state="state" class="space-y-6" @submit="onSubmit">
        <UFormGroup label="Title" name="title">
          <UInput v-model="state.title" class="w-full border rounded px-3 py-2" />
        </UFormGroup>

        <UFormGroup label="Slug" name="slug">
          <UInput v-model="state.slug" class="w-full border rounded px-3 py-2" />
        </UFormGroup>

        <UFormGroup label="Content" name="content">
          <UTextarea v-model="state.content_raw" class="w-full border rounded px-3 py-2 h-32" />
        </UFormGroup>

        <UFormGroup label="Excerpt" name="excerpt">
          <UInput v-model="state.excerpt" class="w-full border rounded px-3 py-2" />
        </UFormGroup>

        <UFormGroup label="Category" name="category_id">
          <USelect v-model="state.category_id" :options="categoryOptions" class="w-full border rounded px-3 py-2" />
        </UFormGroup>

        <UFormGroup class="flex items-center px-3 py-2" label="Is published" name="is_published">
          <UCheckbox class="flex items-center px-3 py-2" v-model="state.is_published" />
        </UFormGroup>
        <div class="flex justify-between">
          <UButton class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded" type="submit">
            Create
          </UButton>
          <nuxt-link class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded" to="/BlogPosts">
            Back
          </nuxt-link>
        </div>
      </UForm>
    </div>
  </div>
</template>

<script setup lang="ts">
import { z } from 'zod';
import { reactive, ref, computed } from 'vue';
import axios from 'axios';

interface Category {
  id: number;
  title: string;
}
interface Post {
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
    console.log(categories.value);
  } catch (error) {
    console.error('Error fetching categories:', error);
  }
};
getCategories();

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

const categoryOptions = computed(() => {
  return categories.value.map(category => ({
    value: category.id.toString(),
    label: category.title
  }));
});

const schema = z.object({
  title: z.string().min(5, 'Title must be at least 5 characters').max(200, 'Title must be less than 200 characters')
      .refine(value => {
        return !posts.value.some(post => post.title === value)
      }, 'Post title must be unique'),
  slug: z.string().max(200, 'Slug must be less than 200 characters')
      .refine(value => {
        return !posts.value.some(post => post.slug === value)
      }, 'Post slug must be unique').optional(),
  content_raw: z.string().min(5, 'Description must be at least 5 characters').max(10000, 'Content must be less than 10000 characters'),
  category_id: z.string().refine(value => {
    const numberValue = Number(value);
    return Number.isInteger(numberValue) && categories.value.some(category => category.id === numberValue);
  }, 'Parent ID must be a valid integer and exist in categories'),
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

async function onSubmit(event: Event) {
  try {
    const result = schema.parse(state);
    console.log(result);

    const dataToSend = {
      ...result,
      category_id: Number(result.category_id)
    };

    const response = await axios.post('http://127.0.0.1:8000/api/blog/posts', dataToSend);
    alert("Post successfully created!");
    window.location.href = '/BlogPosts'
  } catch (e) {
    if (e instanceof z.ZodError) {
      console.error(e.errors);
    } else {
      console.error('Error submitting form:', e);
    }
  }
}
</script>

<style scoped>
.container {
  max-width: 800px;
}
</style>
