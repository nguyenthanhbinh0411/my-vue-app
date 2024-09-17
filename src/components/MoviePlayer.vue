<template>
    <div class="movie-player">
      <div v-if="loading" class="loading-message">
        <p>Đang tải tập phim...</p>
      </div>
      <div v-else>
        <h1 class="movie-title">{{ movie.name }}</h1>
        <iframe v-if="selectedEpisode" :src="selectedEpisode.embed" frameborder="0" allowfullscreen class="video-frame"></iframe>
        <div class="episode-list">
          <h2>Danh sách tập phim:</h2>
          <button 
            v-for="episode in episodes" 
            :key="episode.slug"
            @click="selectEpisode(episode)"
            :class="{ 'active': selectedEpisode && selectedEpisode.slug === episode.slug }"
            class="episode-button"
          >
            Tập {{ episode.name }}
          </button>
        </div>
      </div>
    </div>
  </template>
  
  <script>
  import axios from 'axios';
  
  export default {
    data() {
      return {
        movie: null,
        selectedEpisode: null,
        episodes: [],
        slug: this.$route.params.slug,
        episodeSlug: this.$route.params.episode,
        loading: true,
      };
    },
    methods: {
      async fetchMovieDetails() {
        try {
          const response = await axios.get(`https://phim.nguonc.com/api/film/${this.slug}`);
          if (response.data.status === 'success') {
            this.movie = response.data.movie;
            this.episodes = this.movie.episodes[0].items;
            this.selectedEpisode = this.episodes.find(episode => episode.slug === this.episodeSlug) || this.episodes[0];
          } else {
            console.error('Không tìm thấy phim');
          }
        } catch (error) {
          console.error('Lỗi khi lấy thông tin phim:', error);
        } finally {
          this.loading = false;
        }
      },
      selectEpisode(episode) {
        this.selectedEpisode = episode;
        this.$router.push({ name: 'MoviePlayer', params: { slug: this.slug, episode: episode.slug } });
      },
    },
    mounted() {
      this.fetchMovieDetails();
    },
  };
  </script>
  
  <style scoped>
  .movie-player {
    padding: 20px;
    background-color: #f9f9f9;
    border-radius: 10px;
    max-width: 1200px;
    margin: 20px auto;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  }
  
  .movie-title {
    font-size: 24px;
    margin-bottom: 20px;
  }
  
  .video-frame {
    width: 100%;
    height: 500px;
    border: none;
    border-radius: 8px;
    margin-bottom: 20px;
  }
  
  .loading-message {
    text-align: center;
    font-size: 1.5em;
    color: #888;
  }
  
  .episode-list {
    margin-top: 20px;
  }
  
  .episode-button {
    margin: 5px;
    padding: 10px 15px;
    background-color: #e0e0e0;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  .episode-button.active {
    background-color: #4CAF50;
    color: white;
  }
  
  .episode-button:hover {
    background-color: #d0d0d0;
  }
  </style>