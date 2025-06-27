<template>
  <div class="container">
    <h1 class="main-title">📘 Aplikasi Postingan Mini</h1>

    <h2>📝 Tambah Post</h2>
    <form @submit.prevent="addPost">
      <input v-model="newPost.title" placeholder="Masukkan judul menarik..." required />
      <input v-model="newPost.body" placeholder="Tulis isi postingan di sini..." required />
      <button type="submit">Kirim</button>
    </form>

    <hr />

    <h2>📋 Daftar Post</h2>
    <ul class="post-list">
      <li v-for="post in posts" :key="post.id" class="post-item">
        <div v-if="editedPostId !== post.id">
          <strong>{{ post.title }}</strong> - {{ post.body }}
          <br />
          <small v-if="isValidDate(post.createdAt)">🕒 {{ new Date(post.createdAt).toLocaleString() }}</small>
          <br />
          <button @click="startEdit(post)">Edit</button>
          <button class="delete-btn" @click="deletePost(post.id)">Hapus</button>
        </div>
        <div v-else>
          <input v-model="editPost.title" />
          <input v-model="editPost.body" />
          <button @click="updatePost(post.id)">Simpan</button>
          <button @click="cancelEdit">Batal</button>
        </div>
      </li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'PostApp',
  data() {
    return {
      posts: [],
      newPost: {
        title: '',
        body: ''
      },
      editedPostId: null,
      editPost: {
        title: '',
        body: ''
      }
    }
  },
  mounted() {
    this.loadPosts()
  },
  methods: {
    async loadPosts() {
      try {
        const response = await axios.get('http://localhost:3000/posts')
        this.posts = response.data
      } catch (error) {
        console.error('Gagal mengambil data:', error)
      }
    },
    async addPost() {
      if (!this.newPost.title.trim() || !this.newPost.body.trim()) {
        alert("Judul dan konten tidak boleh kosong!");
        return;
      }
      try {
        await axios.post('http://localhost:3000/posts', {
          ...this.newPost,
          createdAt: new Date().toISOString()
        })
        this.newPost.title = ''
        this.newPost.body = ''
        this.loadPosts()
      } catch (error) {
        console.error('Gagal menambahkan post:', error)
      }
    },
    startEdit(post) {
      this.editedPostId = post.id
      this.editPost = { ...post }
    },
    cancelEdit() {
      this.editedPostId = null
      this.editPost = { title: '', body: '' }
    },
    async updatePost(id) {
      try {
        await axios.put(`http://localhost:3000/posts/${id}`, this.editPost)
        this.editedPostId = null
        this.loadPosts()
      } catch (error) {
        console.error('Gagal memperbarui post:', error)
      }
    },
    async deletePost(id) {
      if (confirm('Yakin ingin menghapus post ini?')) {
        try {
          await axios.delete(`http://localhost:3000/posts/${id}`)
          this.loadPosts()
        } catch (error) {
          console.error('Gagal menghapus post:', error)
        }
      }
    },
    isValidDate(date) {
      return !isNaN(Date.parse(date))
    }
  }
}
</script>

<style>
body {
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(135deg, #74ebd5, #ACB6E5);
  color: #333;
  margin: 0;
  padding: 0;
}

.container {
  max-width: 700px;
  margin: 50px auto;
  padding: 30px;
  background: white;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
  border-radius: 15px;
  animation: fadeIn 0.5s ease-in-out;
}

.main-title {
  text-align: center;
  color: #2c3e50;
  font-size: 2.2rem;
  margin-bottom: 30px;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

input {
  flex: 1 1 40%;
  padding: 10px;
  border-radius: 6px;
  border: 1px solid #ccc;
  transition: all 0.3s ease;
}

input:focus {
  border-color: #7b9acc;
  outline: none;
  box-shadow: 0 0 5px #7b9acc66;
}

button {
  padding: 10px 16px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  background: #6c63ff;
  color: white;
  font-weight: bold;
  transition: all 0.3s ease;
}

button:hover {
  background: #5848d8;
  transform: scale(1.05);
}

button.delete-btn {
  background: #f44336;
}
button.delete-btn:hover {
  background: #d32f2f;
}

.post-list {
  list-style-type: none;
  padding: 0;
}

.post-item {
  background: #ffffffd0;
  margin-bottom: 15px;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease;
}

.post-item:hover {
  transform: translateY(-3px);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(15px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
