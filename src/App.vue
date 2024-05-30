<script setup>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

const form = reactive({
  id: null,
  title: '',
  content: '',
});

const articles = ref([]);

async function load() {
  try {
    const response = await axios.get('http://localhost:3000/articles');
    articles.value = response.data;
  } catch (error) {
    console.error("Error loading articles: ", error);
  }
}

async function save() {
  try {
    const url = form.id
      ? `http://localhost:3000/articles/${form.id}`
      : 'http://localhost:3000/articles';
    const method = form.id ? 'put' : 'post';
    const response = await axios[method](url, form);

    if (form.id) {
      const index = articles.value.findIndex(article => article.id === form.id);
      if (index !== -1) {
        articles.value[index] = response.data;
      }
    } else {
      articles.value.push(response.data);
    }

    form.id = null;
    form.title = '';
    form.content = '';
  } catch (error) {
    console.error('Error saving article: ', error);
  }
}

async function remove(id) {
  try {
    await axios.delete(`http://localhost:3000/articles/${id}`);
    articles.value = articles.value.filter(article => article.id !== id);
  } catch (error) {
    console.error('Error deleting article: ', error);
  }
}

function edit(article) {
  form.id = article.id;
  form.title = article.title;
  form.content = article.content;
}

onMounted(load);
</script>

<template>
  <div class="container">
    <h1>Article Management</h1>
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input"/><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <h3 class="article-title">{{ article.title }}</h3>
        <p class="article-content">{{ article.content }}</p>
        <div class="buttons">
          <button @click="edit(article)" class="button edit-button">Edit</button>
          <button @click="remove(article.id)" class="button delete-button">Delete</button>
        </div>
      </li>
    </ul>
    <button @click="load" class="button load-button">Load</button>
  </div>
</template>

<style scoped>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #333;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.input, .textarea {
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
  font-size: 16px;
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  background-color: #42b883;
  color: white;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.button:hover {
  background-color: #369e6f;
}

.article-list {
  list-style: none;
  padding: 0;
}

.article-item {
  padding: 15px;
  margin-bottom: 10px;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}

.article-title {
  margin: 0;
  color: #333;
}

.article-content {
  margin: 10px 0;
  color: #555;
}

.buttons {
  display: flex;
  gap: 10px;
}

.edit-button {
  background-color: #ffc107;
}

.edit-button:hover {
  background-color: #e0a800;
}

.delete-button {
  background-color: #dc3545;
}

.delete-button:hover {
  background-color: #c82333;
}

.load-button {
  display: block;
  width: 100%;
  margin-top: 20px;
  background-color: #007bff;
}

.load-button:hover {
  background-color: #0056b3;
}
</style>
