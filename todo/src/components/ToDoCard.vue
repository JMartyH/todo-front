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
    <!-- Create To Do Modal -->
    <Modal v-if="showCreateModal" title="Create New To Do" @close="showCreateModal = false">
      <form @submit.prevent="createTodo">
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Title</label>
          <input v-model="newTodo.title" type="text" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Description</label>
          <textarea v-model="newTodo.description" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required></textarea>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Due Date</label>
          <input v-model="newTodo.dueDate" type="datetime-local" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Status</label>
          <select v-model="newTodo.status" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
            <option value="PENDING">Pending</option>
            <option value="COMPLETED">Completed</option>
            <option value="CANCELLED">Cancelled</option>
          </select>
        </div>
        <div class="flex justify-end">
          <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">Create</button>
        </div>
      </form>
    </Modal>
    <!-- Edit To Do Modal -->
    <Modal v-if="showEditModal" title="Edit To Do" @close="showEditModal = false">
      <form @submit.prevent="updateTodo">
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Title</label>
          <input v-model="selectedTodo.title" type="text" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Description</label>
          <textarea v-model="selectedTodo.description" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required></textarea>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Due Date</label>
          <input v-model="selectedTodo.dueDate" type="datetime-local" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline" required>
        </div>
        <div class="mb-4">
          <label class="block text-gray-700 text-sm font-bold mb-2">Status</label>
          <select v-model="selectedTodo.status" class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline">
            <option value="PENDING">Pending</option>
            <option value="COMPLETED">Completed</option>
            <option value="CANCELLED">Cancelled</option>
          </select>
        </div>
        <div class="flex justify-between">
          <button type="button" @click="deleteTodo" class="bg-red-500 text-white px-4 py-2 rounded">Delete</button>
          <button type="submit" class="bg-blue-500 text-white px-4 py-2 rounded">Save</button>
        </div>
      </form>
    </Modal>
    <div>
        HELLO
    <ul v-if="reminders.length > 0">
      <li v-for="reminder in reminders" :key="reminder">
        {{ reminder }}
      </li>
    </ul>
  </div>
  </div>
 
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';
import Modal from './Modal.vue';
import axios from 'axios';
import SockJS from 'sockjs-client';
import { Client } from '@stomp/stompjs';
const reminders = ref([]);
let stompClient = null;

onMounted(() => {
  fetchTodos();
  connectToWebSocket();
});

onUnmounted(() => {
  disconnectFromWebSocket();
});

const connectToWebSocket = () => {
  // Create the SockJS instance
  const socket = new SockJS('http://localhost:8080/ws'); // Your WebSocket endpoint

  stompClient = new Client({
    webSocketFactory: () => socket, // Use SockJS for the connection
    onConnect: () => {
      console.log('Connected to WebSocket (STOMP)');
      stompClient.subscribe('/topic/reminders', (reminder) => {
        reminders.value.push(reminder.body); 
      });
    },
    onError: (error) => {
      console.error('WebSocket connection error:', error);
      // Consider reconnect logic here
    }
  });

  stompClient.activate(); 
};

const disconnectFromWebSocket = () => {
  if (stompClient !== null) {
    stompClient.deactivate(); 
  }
  console.log('Disconnected from WebSocket (STOMP)');
};

const todos = ref([]);
const searchTerm = ref('');
const showCreateModal = ref(false);
const showEditModal = ref(false);
const newTodo = ref({
  title: '',
  description: '',
  dueDate: '',
  status: 'PENDING',
});
const selectedTodo = ref({
  id: null,
  title: '',
  description: '',
  dueDate: '',
  status: 'PENDING',
});
const page = ref(0);
const pageSize = 5;
const filterStatus = ref('');

const fetchTodos = async () => {
  try {
    const response = await axios.get(`http://localhost:8080/api/v1/todos`, {
      params: {
        page: page.value,
        size: pageSize,
        status: filterStatus.value 
      }
    });
    todos.value = response.data; // Axios automatically parses JSON
  } catch (error) {
    console.error('Error fetching todos:', error); 
  }
};

const searchTodos = async () => {
  try {
    const response = await axios.get(`http://localhost:8080/api/v1/todos/search`, {
      params: {
        keyword: searchTerm.value,
        page: page.value,
        size: pageSize,
        status: filterStatus.value
      }
    });
    todos.value = response.data; 
  } catch (error) {
    console.error('Error fetching todos:', error);
  }
};

const createTodo = async () => {
  try {
    const response = await axios.post('http://localhost:8080/api/v1/todos', {
        title: newTodo.value.title,
        description: newTodo.value.description,
        dueDate: new Date(newTodo.value.dueDate).toISOString(),
        status: newTodo.value.status

    });
    if (response.status === 201) {
      fetchTodos();
      showCreateModal.value = false;
      newTodo.value = {
        title: '',
        description: '',
        dueDate: '',
        status: 'PENDING',
      };
    } else {
      console.error('Error creating todo:', response.statusText);
    }
  } catch (error) {
    console.error('Error creating todo:', error);
  }
};

const editTodo = (todo) => {
  selectedTodo.value = { ...todo };
  showEditModal.value = true;
};

const updateTodo = async () => {
  try {
    const response = await axios.put(`http://localhost:8080/api/v1/todos/${selectedTodo.value.id}`, {        
      title: selectedTodo.value.title,
      description: selectedTodo.value.description,
      dueDate: new Date(selectedTodo.value.dueDate).toISOString(),
      status: selectedTodo.value.status,
    });
    if (response.status === 200) {
      fetchTodos();
      showEditModal.value = false;
      selectedTodo.value = {
        id: selectedTodo.value.id,
        title: selectedTodo.value.title,
        description: selectedTodo.value.description,
        dueDate: selectedTodo.value.dueDate,
        status: selectedTodo.value.status,
      };
    } else {
      console.error('Error updating todo:', response.statusText);
    }
  } catch (error) {
    console.error('Error updating todo:', error);
  }
};

const deleteTodo = async () => {
  try {
    const response = await axios.delete(`http://localhost:8080/api/v1/todos/${selectedTodo.value.id}`, {
    });
    if (response.status === 204) {
      fetchTodos();
      showEditModal.value = false;
      selectedTodo.value = {
        id: null,
        title: '',
        description: '',
        dueDate: '',
        status: 'PENDING',
      };
    } else {
      console.error('Error deleting todo:', response.statusText);
    }
  } catch (error) {
    console.error('Error deleting todo:', error);
  }
};

const nextPage = () => {
  if (todos.value.page.totalPages && page.value + 1 < todos.value.page.totalPages) {
    page.value++;
    fetchTodos();
  }
};

const prevPage = () => {
  if (page.value > 0) {
    page.value--;
    fetchTodos();
  }
};

// onMounted(() => {
//   fetchTodos();
// });
</script>