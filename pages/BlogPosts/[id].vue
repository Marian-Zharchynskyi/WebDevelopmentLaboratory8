<template>
  <div class="max-w-4xl mx-auto mt-10">
    <!-- Back button -->
    <div class="mb-6">
      <nuxt-link class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded" to="/BlogPosts">Назад</nuxt-link>
    </div>

    <!-- Main content -->
    <div class="flex flex-col md:flex-row justify-between gap-6">
      <!-- Post details -->
      <div class="w-full md:w-8/12">
        <UCard class="h-full">
          <!-- Post header -->
          <template #header>
            <p class="text-xl font-bold">Post ID: {{ postId }}</p>
          </template>

          <!-- Post details list -->
          <ul class="divide-y divide-gray-200">
            <li class="py-3">
              <span class="font-semibold">Title:</span> {{ post?.title }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Is published:</span> {{ post?.is_published ? 'Так' : 'Ні' }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Published at:</span> {{ post?.published_at }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Created at:</span> {{ post?.created_at }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Excerpt:</span> {{ post?.excerpt }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Slug:</span> {{ post?.slug }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Updated at:</span> {{ post?.updated_at }}
            </li>
          </ul>

          <!-- Post content (in accordion) -->
          <template #footer>
            <UAccordion :items="items">
              <template #content="{ description }">
                <p class="text-justify">{{ post?.content_raw }}</p>
              </template>
            </UAccordion>
          </template>
        </UCard>
      </div>

      <!-- Sidebar: Category and User details -->
      <div class="w-full md:w-4/12">
        <!-- Category card -->
        <UCard>
          <template #header>
            <p class="text-xl font-bold">Категорія</p>
          </template>
          <ul class="divide-y divide-gray-200">
            <li class="py-3">
              <span class="font-semibold">Назва:</span> {{ post?.category?.title }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Опис:</span> {{ post?.category?.description || 'Порожньо' }}
            </li>
          </ul>
        </UCard>

        <!-- User card -->
        <UCard class="mt-6">
          <template #header>
            <p class="text-xl font-bold">Користувач</p>
          </template>
          <ul class="divide-y divide-gray-200">
            <li class="py-3">
              <span class="font-semibold">Ім'я:</span> {{ post?.user?.name }}
            </li>
            <li class="py-3">
              <span class="font-semibold">Електронна пошта:</span> {{ post?.user?.email }}
            </li>
          </ul>
        </UCard>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { useRoute } from 'vue-router';

interface User {
  name: string;
  email: string;
}

interface Category {
  title: string;
  description: string;
}

interface Post {
  id: number;
  user: User;
  category: Category;
  title: string;
  excerpt: string;
  published_at: string;
  created_at: string;
  updated_at: string;
  content_raw: string;
  is_published: boolean;
  slug: string;
}

const route = useRoute();
const postId = ref<number | null>(null);
const post = ref<Post | null>(null);

const getPost = async (id: number) => {
  try {
    const response = await $fetch<Post>(`http://127.0.0.1:8000/api/blog/posts/${id}`);
    post.value = response;
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

const items = [{
  label: 'Content',
  icon: 'i-heroicons-information-circle',
  slot: 'content'
}]
</script>

<style scoped>
/* Add any scoped styles here */
</style>
