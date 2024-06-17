<template>
  <div class="container mx-auto p-4">
    <div class="flex justify-center">
      <div class="w-full">
        <nav class="navbar bg-gray-100 p-4 rounded-lg shadow-sm">
          <a href="/admin/blog/posts/create" class="text-blue-600 font-semibold">Додати</a>
        </nav>
        <div class="card mt-4 bg-white p-4 rounded-lg shadow-md">
          <div class="card-body">
            <table class="table table-auto w-full">
              <thead class="bg-gray-200">
              <tr>
                <th class="border-b-2 border-gray-300 py-2">#</th>
                <th class="border-b-2 border-gray-300 py-2">Автор</th>
                <th class="border-b-2 border-gray-300 py-2">Категорія</th>
                <th class="border-b-2 border-gray-300 py-2">Заголовок</th>
                <th class="border-b-2 border-gray-300 py-2">Дата публікації</th>
              </tr>
              </thead>
              <tbody>
              <tr v-for="post in posts" :key="post.id" class="hover:bg-gray-100">
                <td class="border-b border-gray-300 py-2">{{ post.id }}</td>
                <td class="border-b border-gray-300 py-2">{{ post.user.name }}</td>
                <td class="border-b border-gray-300 py-2">{{ post.category.title }}</td>
                <td class="border-b border-gray-300 py-2">
                  <a :href="'/admin/blog/posts/' + post.id + '/edit'" class="text-blue-500 hover:text-blue-700">{{ post.title }}</a>
                </td>
                <td class="border-b border-gray-300 py-2">{{ post.published_at }}</td>
              </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">

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
</script>

<style scoped>
.navbar {
  background-color: #f0f4f8;
  padding: 1rem;
  border-radius: 8px;
}
.card {
  margin-top: 1rem;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
}
.table th, .table td {
  padding: 0.5rem;
  border-bottom: 1px solid #e2e8f0;
}
.table th {
  background-color: #edf2f7;
}
.table tr:hover {
  background-color: #f7fafc;
}
</style>
