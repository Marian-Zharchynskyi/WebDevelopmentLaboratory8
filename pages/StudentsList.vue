<script setup lang="ts">

const columns = [
  { key: 'title', label: 'Title'},
  { key: 'description', label: 'Description'},
  { key: 'price', label: 'Price'},
  { key: 'rating', label: 'Rating'},
  { key: 'brand', label: 'Brand'},
  { key: 'category', label: 'Category'},
  { key: 'thumbnail', label: 'Thumbnail'}
];

const { data } = await useFetch<any>('https://dummyjson.com/products');
const products = data.value.products;
const q = ref('');
const page = ref(1);
const pageCount = 4;
const total = ref(products.length);

const filteredRows = computed(() => {
  if (!q.value) {
    total.value = products.length;
    return products.slice((page.value - 1) * pageCount, (page.value) * pageCount);
  }
  page.value = 1;
  let result = products.filter((product: any) => {
    return Object.values(product).some(value =>
        String(value).toLowerCase().includes(q.value.toLowerCase())
    );
  });
  total.value = result.length;
  return result.slice((page.value - 1) * pageCount, (page.value) * pageCount);
});

</script>

<template>
  <div>
    <div class="flex px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput v-model="q" placeholder="Filter products..."/>
    </div>
    <UTable class="w-full" :ui="{ td: { base: 'mx-auto max-w-[0] truncate' } }" :columns="columns" :rows="filteredRows">
      <template #thumbnail-data="{ row }">
        <img class="w-[100px] h-[100px]" :src="row.thumbnail" alt="Thumbnail" />
      </template>
      <template #rating-data="{ row }">
        <span :class="row.rating < 4.5 ? 'text-red-700' : 'text-green-700' ">{{ row.rating }}</span>
      </template>
    </UTable>
    <div class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="total" />
    </div>
  </div>
</template>
