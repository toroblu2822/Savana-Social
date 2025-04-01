<!DOCTYPE html>
<html lang="it">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Savana Social</title>
  <script src="https://unpkg.com/vue@3.4.15/dist/vue.global.prod.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet">
  <style>
    body { background-color: #fef9c3; }
    .slide-panel { transition: transform 0.3s ease-in-out; }
    .slide-panel-right.hidden { transform: translateX(100%); }
    .slide-panel-left.hidden { transform: translateX(-100%); }
  </style>
</head>
<body>
<div id="app" class="min-h-screen relative p-6 text-gray-800 font-sans">
  <!-- HEADER -->
  <div class="fixed top-0 left-0 right-0 bg-yellow-500 shadow text-white text-xl font-bold py-3 px-6 z-20 flex justify-between items-center">
    🌿 Savana Social
    <div class="flex space-x-2 items-center">
      <input v-model="searchQuery" placeholder="🔍 Cerca profilo..." class="rounded px-2 py-1 text-sm text-black">
      <button @click="toggleOnlinePanel" class="bg-yellow-600 px-3 py-1 rounded hover:bg-yellow-700 text-sm">
        👥 Online ({{ onlineUsers.length }})
      </button>
    </div>
  </div>

  <!-- PANELLO ONLINE -->
  <div :class="['fixed top-0 right-0 h-full w-64 bg-white shadow-lg slide-panel slide-panel-right z-30', showOnline ? '' : 'hidden']">
    <div class="p-4 border-b font-bold flex justify-between items-center">
      Utenti Online
      <button @click="toggleOnlinePanel" class="text-red-500">✕</button>
    </div>
    <ul class="p-4 space-y-2 text-sm">
      <li v-for="user in onlineUsers" :key="user" class="text-gray-700">🟢 {{ user }}</li>
    </ul>
  </div>

  <!-- PANELLO PROFILO -->
  <div :class="['fixed top-0 left-0 h-full w-72 bg-white shadow-lg slide-panel slide-panel-left z-30 p-4', activeProfile ? '' : 'hidden']">
    <div class="flex justify-between items-center mb-4">
      <h3 class="font-bold">👤 Profilo</h3>
      <button @click="closeProfile" class="text-red-500">✕</button>
    </div>
    <div v-if="activeProfile">
      <img v-if="activeProfile.profilePic" :src="activeProfile.profilePic" class="w-20 h-20 object-cover rounded-full mb-2">
      <h2 class="font-bold text-lg">{{ activeProfile.username }}</h2>
      <p class="text-sm text-gray-600 italic">{{ activeProfile.species }}</p>
      <p class="text-sm mt-2">{{ activeProfile.bio }}</p>
      <div class="mt-2 text-sm">
        <p><strong>Follower:</strong> {{ getFollowersCount(activeProfile.username) }}</p>
        <p><strong>Seguiti:</strong> {{ getFollowingCount(activeProfile.username) }}</p>
        <button v-if="activeProfile.username !== username" @click="toggleFollow(activeProfile.username)" class="mt-2 bg-blue-500 text-white px-3 py-1 rounded text-sm">
          {{ isFollowing(activeProfile.username) ? 'Non seguire' : 'Segui' }}
        </button>
      </div>
      <hr class="my-3">
      <h4 class="font-semibold text-sm mb-1">Post recenti:</h4>
      <div v-for="p in posts.filter(p => p.author === activeProfile.username)" class="text-sm mb-2 border-b pb-2">
        <p>{{ p.content }}</p>
        <img v-if="p.image" :src="p.image" class="w-full max-h-32 object-contain mt-1 rounded">
      </div>
    </div>
  </div>

  <!-- CONTENUTO PRINCIPALE -->
  <div class="max-w-2xl mx-auto pt-24 space-y-6">
    <!-- LOGIN -->
    <div v-if="!loggedIn" class="bg-white p-6 rounded-xl shadow space-y-4">
      <input v-model="username" placeholder="Nome animale" class="w-full p-2 border rounded">
      <select v-model="species" class="w-full p-2 border rounded">
        <option disabled value="">Specie</option>
        <option>Leone</option>
        <option>Giraffa</option>
        <option>Elefante</option>
        <option>Zebra</option>
        <option>Ghepardo</option>
        <option>Iena</option>
        <option>Scimmia</option>
        <option>Antilope</option>
      </select>
      <textarea v-model="bio" placeholder="Bio..." class="w-full p-2 border rounded"></textarea>
      <input type="file" @change="handleProfilePic" class="w-full border rounded p-2">
      <button @click="login" class="w-full bg-yellow-600 text-white p-2 rounded hover:bg-yellow-700">Entra</button>
    </div>

    <!-- CERCA PROFILI -->
    <div v-if="loggedIn && filteredProfiles.length" class="bg-white p-4 rounded-xl shadow space-y-2">
      <h3 class="font-bold text-lg text-yellow-700">🔍 Profili trovati:</h3>
      <div v-for="p in filteredProfiles" :key="p.username" class="border-b pb-2 mb-2 cursor-pointer" @click="openProfile(p)">
        <div class="flex items-center space-x-3">
          <img v-if="p.profilePic" :src="p.profilePic" class="w-10 h-10 rounded-full object-cover">
          <div>
            <p class="font-bold">{{ p.username }} ({{ p.species }})</p>
            <p class="text-sm text-gray-600 italic">{{ p.bio }}</p>
          </div>
        </div>
      </div>
    </div>

    <!-- POST & FEED -->
    <div v-if="loggedIn">
      <div class="bg-white p-4 rounded-xl shadow flex items-center justify-between">
        <div class="flex items-center space-x-3">
          <img v-if="profilePic" :src="profilePic" class="w-12 h-12 rounded-full object-cover">
          <div>
            <h2 class="font-bold">{{ username }} ({{ species }})</h2>
            <p class="text-sm italic">{{ bio }}</p>
          </div>
        </div>
        <button @click="logout" class="text-red-500 text-sm">Esci</button>
      </div>

      <div class="bg-white p-4 rounded-xl shadow space-y-2 mt-4">
        <textarea v-model="newPost" placeholder="Scrivi qualcosa..." class="w-full p-2 border rounded"></textarea>
        <input type="file" @change="handlePostImage" class="w-full border p-2 rounded">
        <button @click="submitPost" class="w-full bg-blue-600 text-white p-2 rounded hover:bg-blue-700">Pubblica</button>
      </div>

      <div v-for="post in posts" :key="post.id" class="bg-white p-4 rounded-xl shadow mt-4 space-y-2">
        <div class="flex items-center space-x-3 cursor-pointer" @click="openProfileByName(post.author)">
          <img v-if="post.avatar" :src="post.avatar" class="w-10 h-10 rounded-full object-cover">
          <div>
            <p class="font-bold">{{ post.author }} ({{ post.species }})</p>
            <p class="text-xs text-gray-500">{{ post.timestamp }}</p>
          </div>
        </div>
        <p>{{ post.content }}</p>
        <img v-if="post.image" :src="post.image" class="rounded max-h-64 object-contain w-full">
        <div class="flex items-center space-x-4 text-sm mt-2">
          <button @click="toggleLike(post)" :class="userLiked(post) ? 'text-pink-500' : 'text-gray-600'">❤️ Like ({{ post.likes.length }})</button>
          <button @click="post.showComments = !post.showComments" class="text-blue-600">💬 Commenti ({{ post.comments.length }})</button>
        </div>
        <div v-if="post.showComments" class="space-y-2 mt-2">
          <div v-for="c in post.comments" class="bg-gray-100 p-2 rounded text-sm">
            <strong>{{ c.author }}:</strong> {{ c.text }}
          </div>
          <div class="flex space-x-2">
            <input v-model="post.newComment" placeholder="Scrivi un commento..." class="flex-1 p-1 text-sm border rounded">
            <button @click="addComment(post)" class="text-sm bg-blue-500 text-white px-2 rounded">Invia</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
const { createApp } = Vue;
createApp({
  data() {
    return {
      username: '',
      species: '',
      bio: '',
      profilePic: '',
      postImage: '',
      loggedIn: false,
      newPost: '',
      posts: JSON.parse(localStorage.getItem('posts')) || [],
      onlineUsers: [],
      allProfiles: JSON.parse(localStorage.getItem('profiles')) || [],
      following: JSON.parse(localStorage.getItem('following')) || {},
      showOnline: false,
      searchQuery: '',
      activeProfile: null
    };
  },
  computed: {
    filteredProfiles() {
      const query = this.searchQuery.toLowerCase();
      if (query === '') return [];
      return this.allProfiles.filter(p =>
        p.username.toLowerCase().includes(query) ||
        p.species.toLowerCase().includes(query)
      ).filter(p => p.username !== this.username); 
    }
  },
  methods: {
    filterText(text) {
      const badWords = ['parola1', 'parola2', 'parola3']; 
      const sillyWord = 'banana'; 
      let clean = text;
      badWords.forEach(bad => {
        const regex = new RegExp(`\\b${bad}\\b`, 'gi');
        clean = clean.replace(regex, sillyWord);
      });
      return clean;
    },

    clearLocalStorage() {
      localStorage.clear(); // Cancella tutti i dati del localStorage
    },

    login() {
      this.clearLocalStorage(); // Pulisce il localStorage prima del login
      if (!this.username || !this.species) return alert("Compila tutti i campi!");
      this.username = this.filterText(this.username);
      this.bio = this.filterText(this.bio);
      this.loggedIn = true;
      if (!this.onlineUsers.includes(this.username)) this.onlineUsers.push(this.username);
      const exists = this.allProfiles.some(p => p.username === this.username);
      if (!exists) {
        const newProfile = {
          username: this.username,
          species: this.species,
          bio: this.bio,
          profilePic: this.profilePic
        };
        this.allProfiles.push(newProfile);
        localStorage.setItem('allProfiles', JSON.stringify(this.allProfiles));
      }
      localStorage.setItem('onlineUsers', JSON.stringify(this.onlineUsers));
    },

    logout() {
      this.loggedIn = false;
      this.onlineUsers = this.onlineUsers.filter(u => u !== this.username);
      this.username = this.species = this.bio = this.profilePic = '';
      this.activeProfile = null;
    },

    toggleOnlinePanel() {
      this.showOnline = !this.showOnline;
    },

    handleProfilePic(e) {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = e => this.profilePic = e.target.result;
        reader.readAsDataURL(file);
      }
    },

    handlePostImage(e) {
      const file = e.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = e => this.postImage = e.target.result;
        reader.readAsDataURL(file);
      }
    },

    submitPost() {
      if (!this.newPost && !this.postImage) return;
      const filteredContent = this.filterText(this.newPost);
      const newPost = {
        id: Date.now(),
        author: this.username,
        species: this.species,
        avatar: this.profilePic,
        content: filteredContent,
        image: this.postImage || '',
        timestamp: new Date().toLocaleString(),
        likes: [],
        comments: [],
        showComments: false,
        newComment: ''
      };
      this.posts.unshift(newPost);
      localStorage.setItem('posts', JSON.stringify(this.posts));
      this.newPost = '';
      this.postImage = '';
    },

    toggleLike(post) {
      const index = post.likes.indexOf(this.username);
      if (index === -1) post.likes.push(this.username);
      else post.likes.splice(index, 1);
      localStorage.setItem('posts', JSON.stringify(this.posts));
    },

    userLiked(post) {
      return post.likes.includes(this.username);
    },

    addComment(post) {
      const text = this.filterText(post.newComment.trim());
      if (text) {
        post.comments.push({ author: this.username, text: text });
        post.newComment = '';
        localStorage.setItem('posts', JSON.stringify(this.posts));
      }
    },

    openProfile(p) {
      this.activeProfile = p;
    },

    openProfileByName(name) {
      const p = this.allProfiles.find(p => p.username === name);
      if (p) this.activeProfile = p;
    },

    closeProfile() {
      this.activeProfile = null;
    },

    toggleFollow(user) {
      if (!this.following[this.username]) this.following[this.username] = [];
      const idx = this.following[this.username].indexOf(user);
      if (idx === -1) this.following[this.username].push(user);
      else this.following[this.username].splice(idx, 1);
      localStorage.setItem('following', JSON.stringify(this.following));
    },

    isFollowing(user) {
      return this.following[this.username]?.includes(user);
    },

    getFollowersCount(user) {
      return Object.values(this.following).filter(list => list.includes(user)).length;
    },

    getFollowingCount(user) {
      return this.following[user]?.length || 0;
    },

    saveProfiles() {
      localStorage.setItem('profiles', JSON.stringify(this.allProfiles));
    }
  }
}).mount('#app');
</script>
</body>
</html>
