<template>
  <div>
    <h1>Anime</h1>
    <div v-if="loading" class="loading">Đang tải...</div>
    <div v-else>
      <div class="container">
        <div class="movie-list">
          <div v-for="movie in movies" :key="movie.slug" class="movie-item">
            <img :src="movie.thumb_url" :alt="movie.name" class="movie-thumb" />
            <h2>{{ movie.name }}</h2>
            <p>Chất lượng: {{ movie.quality }}</p>
            <p>Lượt xem: {{ formatViews(movie.views) }}</p> 
            <router-link :to="{ name: 'MovieDetails', params: { slug: movie.slug } }" class="movie-link">Xem phim</router-link>
          </div>
        </div>
      </div>
      <div class="pagination">
        <button @click="prevPage" :disabled="page <= 1">Trang trước</button>
        <span>Trang {{ page }} / {{ totalPages }}</span>
        <button @click="nextPage" :disabled="page >= totalPages">Trang sau</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'MovieList',
  data() {
    return {
      movies: [],
      page: 1,
      totalPages: 1,
      loading: true,
    };
  },
  methods: {
    async fetchMovies(page) {
      // Xóa dữ liệu localStorage khi chuyển trang
      localStorage.clear();

      try {
        this.loading = true;
        const responses = await Promise.all([
          axios.get(`https://phim.nguonc.com/api/films/the-loai/hoat-hinh?page=${page}`),
          axios.get(`https://phim.nguonc.com/api/films/the-loai/hoat-hinh?page=${page + 1}`)
        ]);

        const [responsePage1, responsePage2] = responses;
        const moviesPage1 = responsePage1.data.items;
        const moviesPage2 = responsePage2.data.items;

        this.movies = [...moviesPage1, ...moviesPage2].map(movie => {
          let storedViews = localStorage.getItem(`movieViews-${movie.slug}`);
          if (!storedViews) {
            storedViews = Math.floor(Math.random() * 990000) + 10000; 
            localStorage.setItem(`movieViews-${movie.slug}`, storedViews);
          }
          return {
            ...movie,
            views: parseInt(storedViews, 10)
          };
        });
        this.page = responsePage1.data.paginate.current_page;
        this.totalPages = responsePage1.data.paginate.total_page;
      } catch (error) {
        console.error("Lỗi khi tải dữ liệu:", error);
      } finally {
        this.loading = false;
      }
    },
    prevPage() {
      if (this.page > 1) {
        this.page--;
        this.fetchMovies(this.page);
      }
    },
    nextPage() {
      if (this.page < this.totalPages) {
        this.page++;
        this.fetchMovies(this.page);
      }
    },
    formatViews(views) {
      return views.toLocaleString(); // Thêm dấu phẩy ngăn cách hàng nghìn
    },
  },
  created() {
    this.fetchMovies(this.page);
  },
};
</script>

<style scoped>
.container {
max-width: 1200px; /* Set maximum width of the container */
margin: 0 auto; /* Center the container */
padding: 0 16px; /* Add some horizontal padding */
}

.movie-list {
display: flex;
flex-wrap: wrap;
gap: 16px;
justify-content: center;
}

.movie-item {
width: calc(25% - 16px); /* Adjust width to fit 4 items per row */
border: 1px solid #ddd;
padding: 8px; /* Reduce padding for a smaller item */
text-align: center;
box-sizing: border-box; /* Ensure padding and border are included in the width */
transition: transform 0.3s, box-shadow 0.3s; /* Smooth transition for effects */
}

.movie-item:hover {
transform: scale(1.05); /* Slightly enlarge the item on hover */
box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2); /* Add shadow on hover */
}

.movie-thumb {
width: 200px; /* Reduce width of images */
height: 280px; /* Reduce height of images */
object-fit: cover; /* Ensure image covers the container without distortion */
margin-bottom: 8px; /* Add some space below the image */
}

.movie-item h2 {
font-size: 14px; /* Adjust font size for title */
margin: 8px 0; /* Add space above and below the title */
}

.movie-item p {
font-size: 12px; /* Adjust font size for quality text */
margin: 8px 0; /* Add space above and below the quality text */
}

.movie-link {
display: inline-block;
margin-top: 8px;
padding: 8px 16px;
background-color: #007bff;
color: white;
text-decoration: none;
border-radius: 4px;
transition: background-color 0.3s; /* Smooth transition for button color */
}

.movie-link:hover {
background-color: #0056b3; /* Darker shade on hover */
}

.pagination {
margin-top: 16px;
text-align: center;
}

.loading {
text-align: center;
font-size: 18px;
margin-top: 20px;
}
</style>