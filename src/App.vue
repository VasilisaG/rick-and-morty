<!-- App.vue -->
<template>
  <div id="app">
    <header>
      <h1>Rick and Morty Characters</h1>
    </header>
    <main>
      <div class="nev">
        <input type="text" v-model="searchQuery" placeholder="Search by name">
        <select v-model="selectedStatus">
          <option value="">All Statuses</option>
          <option value="alive">Alive</option>
          <option value="dead">Dead</option>
          <option value="unknown">Unknown</option>
        </select>
        <button @click="applyFilters">Apply Filters</button>
      </div>
      <div v-if="loading">Loading...</div>
      <div v-else>
        <div class="cards">
          <div v-for="character in filteredCharacters" :key="character.id" class="card">
            <img :src="character.image" alt="Character Image" class="card-image">
            <div class="card-content">
              <h2>{{ character.name }}</h2>
              <p>Status: {{ character.status }}</p>
              <p>Species: {{ character.species }}</p>
              <p>Last Location: {{ character.lastLocation }}</p> 
              <p>First Episode: {{ character.firstEpisode }}</p> 
              <p>Episodes: {{ character.episodes.join(', ') }}</p> <!-- Добавлено отображение эпизодов -->
            </div>
          </div>
        </div>
        <div class="pagination">
          <button :disabled="currentPage === 1" @click="changePage('prev')">Prev</button>
          <button :disabled="currentPage === totalPages" @click="changePage('next')">Next</button>
          <span>Page {{ currentPage }} of {{ totalPages }}</span>
        </div>
      </div>
    </main>
  </div>
</template>


<script>
export default {
  data() {
    return {
      characters: [],
      filteredCharacters: [],
      loading: false,
      currentPage: 1,
      totalPages: 1,
      searchQuery: '',
      selectedStatus: ''
    };
  },
  created() {
    this.fetchCharacters();
  },
  methods: {
    async fetchCharacters() {
      this.loading = true;
      try {
        const queryParams = new URLSearchParams({
          page: this.currentPage.toString(),
          name: this.searchQuery, 
          status: this.selectedStatus, 
        });
        const response = await fetch(`https://rickandmortyapi.com/api/character?${queryParams}`);
        const data = await response.json();
        this.characters = await Promise.all(data.results.map(async character => {
          const episodes = await Promise.all(character.episode.map(async episodeUrl => {
            const episodeResponse = await fetch(episodeUrl);
            return await episodeResponse.json();
          }));

          return {
            ...character,
            image: character.image,
            lastLocation: character.location.name,
            episodes: episodes.map(episode => episode.name)
          };
        }));
        this.filteredCharacters = this.characters;
        this.totalPages = data.info.pages;
      } catch (error) {
        console.error('Error fetching characters:', error);
      } finally {
        this.loading = false;
      }
    },

    applyFilters() {
      this.currentPage = 1;
      this.fetchCharacters();
    },
    changePage(direction) {
     if (direction === 'next' && this.currentPage < this.totalPages) {
       this.currentPage++;
     } else if (direction === 'prev' && this.currentPage > 1) {
       this.currentPage--;
     }
     this.fetchCharacters();
    } 
  }
};
</script>
<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  font-family: Arial, sans-serif;
  background-color: #f0f0f0;
}

#app {
  margin: 0 auto;
  padding-bottom: 20px;
}

header {
  text-align: center;
  margin-bottom: 20px;
  background-size: cover;
  background-position: center;
  color: black;
  padding: 100px 0; 
}

header h1 {
  
  margin-bottom: 10px;
}

.nev {
  top: 40%;
  position: absolute; 
  left: 50%; 
  transform: translate(-50%, -50%); 
  text-align: center; 
}


input[type="text"],
select {
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
  margin-right: 10px;
}

h1 {
  font-size: 36px;
  margin-bottom: 10px;
  color: rgb(32, 35, 41);
  font-size: 3.125rem;
  font-family: -apple-system, 'BlinkMacSystemFont', 'Segoe UI', 'Roboto', 'Helvetica', 'Arial', sans-serif, 'Apple Color Emoji', 'Segoe UI Emoji', 'Segoe UI Symbol';
  font-weight: 800;
}



input[type="text"],
select {
  padding: 8px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: #98a716;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

button:hover {
  background-color: #45a049;
}

main {
  width: 100%; 
  max-width: 100%; 
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  background:rgb(32, 35, 41);
  padding: 0 20px; 
}


.cards {
  display: flex;
  -webkit-box-pack: center;
  justify-content: center;
  -webkit-box-align: center;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  margin-top: 7%;
}

.card {
  width: 600px;
  height: 220px;
  display: flex;
  overflow: hidden;
  background: rgb(60, 62, 68);
  border-radius: 0.5rem;
  margin: 0.75rem;
  box-shadow: rgba(0, 0, 0, 0.1) 0px 4px 6px -1px, rgba(0, 0, 0, 0.06) 0px 2px 4px -1px;
  margin-bottom: 20px;
  margin-left: auto;
}

.card-image { 
  height: 100%;
  border-radius: 8px;
  margin-bottom: 10px;
  flex: 2 1 0%;
}

.card-content {
  color: #f0f0f0;
  flex: 0 0 60%; 
  padding: 20px;
}
.card h2 {
  font-size: 18px;
  margin-bottom: 5px;
  margin-top: 10px;
  font-weight: bold;
}


.card p {
  font-size: 16px;
  margin-bottom: 10px;
}


.pagination {
  margin-top: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.pagination button {
  padding: 8px 16px;
  font-size: 16px;
  background-color: #f2f2f2;
  color: #4caf50;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
}

.pagination button:hover {
  background-color: #e6e6e6;
}

.pagination button:disabled {
  background-color: #cccccc;
  color: #666666;
  cursor: not-allowed;
}

.pagination span {
  font-size: 16px;
}
@media (max-width: 768px) {
  .card {
    flex-direction: column; 
  }
}

  .card-image {
    flex: 0 0 auto; 
    margin-bottom: 20px; 
  }

  .card-content {
    flex: 0 0 auto;
}
</style>