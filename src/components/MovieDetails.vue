<template>
  <div v-if="movie" class="movie-details">
    <div class="movie-container">
      <div class="movie-info-section">
        <img :src="movie.poster_url" alt="Movie Poster" class="movie-poster"/>
        <div class="movie-info">
          <h1 class="movie-title" style="margin: 0px;">{{ movie.name }}</h1>
          <p><strong>Tên gốc:</strong> {{ movie.original_name }}</p>
          <p><strong>Ngày phát hành:</strong> {{ new Date(movie.created).toLocaleDateString('vi-VN') }}</p>
          <p><strong>Chất lượng:</strong> {{ movie.quality }}</p>
          <p><strong>Ngôn ngữ:</strong> {{ movie.language }}</p>
          <p><strong>Quốc gia:</strong> {{ movie.category[4].list[0].name }}</p>
          <p><strong>Thể loại:</strong> {{ movie.category[2].list[0].name }}</p>
          <p><strong>Số tập:</strong> {{ movie.total_episodes }}</p>
          <p><strong>Tập hiện tại:</strong> {{ movie.current_episode }}</p>
          <p><strong>Lượt xem:</strong> {{ formatViews(movie.views) }}</p> 
          <!-- Hiển thị lượt xem -->
          <p class="movie-description"><strong>Mô tả:</strong>{{ movie.description }}</p>
          <button @click="redirectToMoviePlayer" class="watch-button">Xem phim</button>
        </div>
      </div>
    </div>
  </div>
  <div v-else class="loading-message">
    <p>Đang tải thông tin phim...</p>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      movie: null,
      selectedEpisode: null,
      slug: this.$route.params.slug,  // Lấy slug từ route params
    };
  },
  methods: {
    async fetchMovieDetails() {
      try {
        const response = await axios.get(`https://phim.nguonc.com/api/film/${this.slug}`);
        if (response.data.status === 'success') {
          // Lấy lượt xem từ localStorage
          const storedViews = localStorage.getItem(`movieViews-${this.slug}`);
          this.movie = {
            ...response.data.movie,
            views: storedViews ? parseInt(storedViews, 10) : 0
          };
          this.selectedEpisode = this.movie.episodes[0].items[0];  // Mặc định chọn tập 1
        } else {
          console.error('Không tìm thấy phim');
        }
      } catch (error) {
        console.error('Lỗi khi lấy thông tin phim:', error);
      }
    },
    redirectToMoviePlayer() {
      this.$router.push({ name: 'MoviePlayer', params: { slug: this.slug, episode: this.selectedEpisode.slug } });
    },
    formatViews(views) {
      return views.toLocaleString(); // Thêm dấu phẩy ngăn cách hàng nghìn
    },
  },
  mounted() {
    this.fetchMovieDetails();
  },
};
</script>

<style scoped>
.movie-details {
  padding: 20px;
  background-color: #f9f9f9;
  border-radius: 10px;
  max-width: 1200px;
  margin: 20px auto;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.movie-container {
  display: flex;
  flex-direction: column;
}

.movie-info-section {
  display: flex;
  justify-content: flex-start;
  align-items: flex-start;
}

.movie-info-section .movie-poster {
  width: 300px;
  height: 450px;
  object-fit: cover;
  border-radius: 8px;
}

.movie-info {
  flex: 1;
  padding-left: 20px;
  text-align: left;
}

.movie-title {
  font-size: 2.2em;
  color: #333;
}

.movie-info p {
  font-size: 1.1em;
  color: #555;
  margin: 10px 0;
}

.movie-description {
  font-size: 1.1em;
  color: #555;
}

.watch-button {
  background-color: #1e90ff;
  color: white;
  border: none;
  padding: 10px 20px;
  font-size: 1em;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
}

.watch-button:hover {
  background-color: #0056b3;
}

.loading-message {
  text-align: center;
  font-size: 1.5em;
  color: #888;
}
</style>