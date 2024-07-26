<template>
  <div class="max-w-md mx-auto bg-white shadow-lg rounded-lg overflow-hidden my-4">
    <div class="px-6 py-4">
      <div class="flex gap-10 justify-between items-center">
        <div class="font-bold text-xl">To Do List</div>
        <button @click="showCreateModal = true" class="bg-blue-500 text-white px-3 py-1 rounded">Add To Do</button>
      </div>
      <div class="mb-4">
        <input 
          type="text" 
          v-model="searchTerm"
          @input="searchTodos"
          placeholder="Search..."
          class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
      </div>
      <div class="mb-4">
        <label class="block text-gray-700 text-sm font-bold mb-2">Filter by Status</label>
        <select v-model="filterStatus" @change="fetchTodos" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
          <option value="">All</option>
          <option value="PENDING">Pending</option>
          <option value="COMPLETED">Completed</option>
          <option value="CANCELLED">Cancelled</option>
        </select>
      </div>
      <ul>
        <li v-for="todo in todos.content" :key="todo.id" class="flex flex-col p-2 border-b border-gray-200 cursor-pointer" @click="editTodo(todo)">
          <div class="font-semibold text-lg">{{ todo.title }}</div>
          <div class="text-gray-600">{{ todo.description }}</div>
          <div class="text-sm text-gray-500">Due: {{ new Date(todo.dueDate).toLocaleString() }}</div>
          <div :class="{'text-green-500': todo.status === 'COMPLETED', 'text-red-500': todo.status === 'PENDING', 'text-gray-500': todo.status === 'CANCELLED'}">
            {{ todo.status }}
          </div>
        </li>
      </ul>
      <div class="flex justify-between items-center mt-4">
        <button @click="prevPage" :disabled="page === 0" class="bg-gray-500 text-white px-3 py-1 rounded">Previous</button>
        <span>Page {{ page + 1 }}</span>
        <button @click="nextPage" :disabled="todos.totalPages && page + 1 >= todos.totalPages" class="bg-gray-500 text-white px-3 py-1 rounded">Next</button>
      </div>
    </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const todos = ref([]);
const searchTerm = ref('');
const page = ref(0);
const pageSize = 7;
const filterStatus = ref('');


const fetchTodos = async () => {
  try {
    const response = await fetch(`http://localhost:8080/api/v1/todos?page=${page.value}&size=${pageSize}&status=${filterStatus.value}`);
    const data = await response.json();
    todos.value = data;
  } catch (error) {
    console.error('Error fetching todos:', error);
  }
};

const searchTodos = async () => {
  try {
    const response = await fetch(`http://localhost:8080/api/v1/todos/search?keyword=${searchTerm.value}&page=${page.value}&size=${pageSize}&status=${filterStatus.value}`);
    const data = await response.json();
    todos.value = data;
  } catch (error) {
    console.error('Error fetching todos:', error);
  }
};

const nextPage = () => {
  if (todos.value.page.totalPages && page.value + 1 < todos.value.page.totalPages) {
    page.value++;
    searchTerm.value ? searchTodos() : fetchTodos(); 
  }
};

const prevPage = () => {
  if (page.value > 0) {
    page.value--;
    searchTerm.value ? searchTodos() : fetchTodos();
  }
};

onMounted(() => {
  fetchTodos();
});
</script>