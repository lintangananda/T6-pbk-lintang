<template>
  <div class="container">
    <form @submit.prevent="save" class="form">
      <input type="text" v-model="form.title" placeholder="Title" class="input" /><br />
      <textarea v-model="form.content" placeholder="Content" class="textarea"></textarea><br />
      <button type="submit" class="button">Save</button>
    </form>
    <button @click="load" class="button load-button">Load Articles</button>
    <ul class="article-list">
      <li v-for="article in articles" :key="article.id" class="article-item">
        <div class="article-content">
          <h3>{{ article.title }}</h3>
          <p>{{ article.content }}</p>
        </div>
        <div class="article-actions">
          <button @click="edit(article)" class="button edit-button">Edit</button>
          <button @click="remove(article.id)" class="button delete-button">Delete</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import { ref, onMounted, reactive } from 'vue';
import axios from 'axios';

export default {
  setup() {
    const form = reactive({
      id: '',
      title: '',
      content: '',
    });

    const articles = ref([]);

    async function load() {
      try {
        const response = await axios.get('http://localhost:3000/articles');
        articles.value = response.data;
      } catch (error) {
        console.error('Error loading articles:', error);
      }
    }

    async function save() {
      try {
        if (!form.title || !form.content) {
          alert('Title and Content are required');
          return;
        }

        if (!form.id) {
          form.id = Math.random().toString(36).substring(2, 15);
          const response = await axios.post('http://localhost:3000/articles', form);
          articles.value.push(response.data);
        } else {
          const response = await axios.put(`http://localhost:3000/articles/${form.id}`, form);
          const index = articles.value.findIndex(article => article.id === form.id);
          articles.value[index] = response.data;
        }

        // Reset form
        form.id = '';
        form.title = '';
        form.content = '';
      } catch (error) {
        console.error('Error saving article:', error);
      }
    }

    async function remove(id) {
      try {
        await axios.delete(`http://localhost:3000/articles/${id}`);
        articles.value = articles.value.filter(article => article.id !== id);
      } catch (error) {
        console.error('Error deleting article:', error);
      }
    }

    function edit(article) {
      form.id = article.id;
      form.title = article.title;
      form.content = article.content;
    }

    onMounted(load);

    return { form, articles, save, edit, remove, load };
  }
};
</script>

<style>
.container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  background-color: #ffe6e6;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  border-radius: 8px;
}

.form {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 20px;
}

.input, .textarea {
  width: 100%;
  padding: 10px;
  border: 1px solid #ffccd5;
  border-radius: 4px;
  background-color: #fff0f1;
}

.input:focus, .textarea:focus {
  border-color: #ff99aa;
  outline: none;
  box-shadow: 0 0 5px rgba(255, 153, 170, 0.5);
}

.button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  background-color: #ff80a5;
  color: white;
  cursor: pointer;
  font-size: 16px;
}

.button:hover {
  background-color: #ff4d75;
}

.load-button {
  background-color: #a168f8;
  margin-bottom: 20px;
  width: 100%;
}

.load-button:hover {
  background-color: #7735db;
}

.article-list {
  list-style-type: none;
  padding: 0;
}

.article-item {
  border: 1px solid #ffccd5;
  border-radius: 4px;
  padding: 15px;
  margin-bottom: 10px;
  display: flex;
  flex-direction: column;
  background-color: #fff0f1;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.article-item:hover {
  transform: scale(1.02);
}

.article-content {
  margin-bottom: 10px;
}

.article-content h3 {
  margin: 0 0 10px 0;
}

.article-content p {
  margin: 0;
  color: #555;
}

.article-actions {
  display: flex;
  gap: 10px;
}

.delete-button {
  background-color: #ff4d75;
}

.delete-button:hover {
  background-color: #ff1a47;

}

.edit-button {
  background-color: rgb(118, 193, 118);
}

.edit-button:hover {
  background-color: #02f64f;
}
</style>
