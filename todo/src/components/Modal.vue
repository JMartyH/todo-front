<!-- src/components/Modal.vue -->
<template>
    <div class="fixed inset-0 flex items-center justify-center bg-gray-800 bg-opacity-50">
      <div class="bg-white p-6 rounded-lg shadow-lg w-1/2">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-bold">{{ title }}</h2>
          <button @click="closeModal" class="text-gray-500">&times;</button>
        </div>
        <slot></slot>
        <div v-if="$slots.footer" class="mt-4">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { defineEmits, defineProps } from 'vue';
  
  const props = defineProps({
    title: String,
  });
  
  const emit = defineEmits(['close']);
  
  const closeModal = () => {
    emit('close');
  };
  </script>
  