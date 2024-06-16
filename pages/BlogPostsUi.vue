<script setup lang="ts">
const columns = [
  {key: 'id', label: '#'},
  {key: 'user_name', label: 'Автор'},
  {key: 'category_title', label: 'Категорія'},
  {key: 'title', label: 'Заголовок'},
  {key: 'published_at', label: 'Дата публікації'},
]
const page = ref(1);
const page_count = 5;

interface User {
  name: string;
}

interface Category {
  title: string;
}

interface Post {
  id: number;
  user: User;
  category: Category;
  title: string;
  published_at: string;
}

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

const rows = computed(() => {
  return posts.value.map((post) => ({
    id: post.id,
    user_name: post.user.name,
    category_title: post.category.title,
    title: post.title,
    published_at: post.published_at,
  }));
});

const total = computed(() => rows.value.length);
const paginated_rows = computed(() => {
      return rows.value.slice((page.value - 1) * page_count, (page.value) * page_count);
    }
);

</script>

<template>
  <UTable :columns="columns" :rows="paginated_rows" :total="rows.length" />
  <div>
    <UPagination v-model="page" :page-count="page_count" :total="total" />
  </div>
</template>