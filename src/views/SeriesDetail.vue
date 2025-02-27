<template>
    <div class="series-detail">
      <div v-if="series" class="hero-section">
        <div class="container">
          <div class="row align-items-center">
            <div class="col-md-6 mb-4 mb-md-0">
              <div class="video-container">
                <iframe
                  :src="getEmbedUrl(series.trailer)"
                  allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture"
                  allowfullscreen
                  @load="iframeLoaded"
                ></iframe>
              </div>
              <br>
              <router-link
                v-if="series.fullplaylist"
                :to="{
                  name: 'WatchParty',
                  params: {
                    playlistId: extractPlaylistId(series.fullplaylist),
                    title: series.title.replace(/\s+/g, ' '), // Replace spaces with hyphens for URL
                  },
                }"
                class="playlist-link mt-2 d-inline-block"
              >
                Watch Full Playlist
              </router-link>
              <p v-else class="text-muted mt-2">No playlist available.</p>
            </div>
            <div class="col-md-6">
              <div class="text-start">
                <h1 class="display-4 mb-4 fw-bold text-dark">{{ series.title }}</h1>
                <p class="lead mb-4 text-muted" style="font-size: 1.25rem;">
                  {{ series.shortDescription }}
                </p>
              </div>
  
              <div class="details mb-4 text-start">
                <p class="mb-2 text-dark"><strong>Release Date:</strong> {{ series.releaseDate }}</p>
                <p class="mb-2 text-dark"><strong>Genre:</strong> {{ series.genre.join(", ") }}</p>
              </div>
  
              <div class="synopsis mb-4 text-start">
                <h2 class="h4 fw-bold mb-3 text-dark">Synopsis</h2>
                <p class="text-dark">{{ series.synopsis }}</p>
              </div>
            </div>
          </div>
        </div>
      </div>
      
  
      <div v-else class="not-found">
        <div class="content">
          <h1 class="title">403</h1>
          <p class="subtitle">Oops! Series Not Found</p>
          <p class="description">
            The series you're looking for doesn't exist or has been moved.
          </p>
          <router-link to="/recommendations" class="home-link">
            <i class=""></i> Back to Series
          </router-link>
        </div>
      </div>

        <!-- Scroll to Top Button -->
    <button v-if="showScrollToTop" @click="scrollToTop" class="scroll-to-top">
      <i class="fas fa-arrow-up"></i>
    </button>

    </div>
  </template>




<script>
import seriesData from '@/assets/series.json'; // Import the JSON file

export default {
  name: 'SeriesDetail',
  props: ['id'],
  data() {
    return {
      series: null, // Initialize as null
      userHasPlan: false,
      isLoading: true, // Add a loading state
    };
  },
  created() {
    this.fetchSeriesDetail();
    this.checkUserPlan();
  },
  methods: {
    fetchSeriesDetail() {
      this.isLoading = true; // Start loading

      // Check if seriesData is valid
      if (!Array.isArray(seriesData)) {
        console.error('seriesData is not a valid array');
        this.isLoading = false;
        return;
      }

      // Check if this.id is a valid number
      const seriesId = parseInt(this.id);
      if (isNaN(seriesId)) {
        console.error('Invalid series ID:', this.id);
        this.isLoading = false;
        return;
      }

      // Find the series with the matching ID
      this.series = seriesData.find(series => series.id === seriesId);

      // Check if the series exists
      if (!this.series) {
        console.error('Series not found with ID:', seriesId);
        this.isLoading = false;
        return;
      }

      // Check if the series requires a plan and the user doesn't have one
      if (this.series.requiresPlan && !this.userHasPlan) {
        this.$router.push({ path: '/pricing' });
      }

      this.isLoading = false; // End loading
    },
    checkUserPlan() {
      const userPlan = localStorage.getItem('userPlan'); // Check if the user has a plan
      this.userHasPlan = !!userPlan; // Set to true if the user has a plan
    },
    getEmbedUrl(url) {
      if (!url) return '';
      if (url.includes('watch?v=')) {
        return url.replace('watch?v=', 'embed/');
      }
      return url;
    },
    extractPlaylistId(url) {
      // Extract the playlist ID from the URL
      const match = url.match(/list=([^&]+)/);
      return match ? match[1] : '';
    },
    iframeLoaded() {
      console.log('Iframe loaded successfully');
    },
    handleScroll() {
      this.showScrollToTop = window.scrollY > 200;
    },
    scrollToTop() {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    },
  },
};
</script>

  <style scoped>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600;700&display=swap');
  
  .series-detail {
    font-family: 'Poppins', sans-serif;
    background-color: #ffffff;
    min-height: 100vh; /* Ensure the container takes up the full viewport height */
    display: flex;
    flex-direction: column;
  }
  
  .hero-section {
    flex: 1; /* Take up remaining space */
    padding: 80px 0;
    background-color: #f8f9fa;
    color: #333;
  }
  
  .video-container {
    position: relative;
    padding-bottom: 70%;
    height: 0;
    overflow: hidden;
    border-radius: 10px;
    box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
    max-width: 100%;
    margin: 0 auto;
  }
  
  .video-container iframe {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    border: none;
  }
  
  .playlist-link {
    color: #2c3e50;
    text-decoration: none;
    font-weight: 600;
    transition: color 0.3s ease;
  }
  
  .playlist-link:hover {
    color: #3498db;
  }
  
  h1 {
    font-size: 2.5rem;
    font-weight: 700;
    line-height: 1.3;
    color: #2c3e50;
  }
  
  .lead {
    font-size: 1.25rem;
    line-height: 1.6;
    color: #555;
  }
  
  .details p {
    font-size: 1rem;
    color: #555;
  }
  
  .synopsis {
    margin-bottom: 2rem;
  }
  
  @media (max-width: 768px) {
    h1 {
      font-size: 2rem;
    }
  
    .lead {
      font-size: 1rem;
    }
  
    .details p {
      font-size: 0.9rem;
    }
  
    .hero-section .row {
      flex-direction: column;
    }
  
    .hero-section .col-md-6 {
      width: 100%;
      margin-bottom: 20px;
    }
  }

 /* Base Styles */
.not-found {
  display: flex;
  flex: 1;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
  background: rgb(255, 255, 255); /* Light grey gradient */
  color: #1f2937; /* Neutral dark color for text */
  font-family: 'Inter', sans-serif;
  padding: 20px;
  animation: fadeIn 0.8s ease-in-out;
  margin-top: -12%; /* Added margin-top for mobile */
}

.content {
  text-align: center;
  max-width: 600px;
}

.title {
  font-size: clamp(6rem, 15vw, 10rem); /* Responsive font size */
  font-weight: 900;
  margin: 0;
  background: linear-gradient(45deg, #6366f1, #38bdf8); /* Gradient for title text */
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: float 3s ease-in-out infinite;
}

.subtitle {
  font-size: clamp(1.5rem, 5vw, 2.5rem); /* Responsive font size */
  font-weight: 600;
  margin: 10px 0;
  color: #4b5563; /* Neutral color for subtitle */
}

.description {
  font-size: clamp(1rem, 4vw, 1.5rem); /* Responsive font size */
  margin: 20px 0;
  color: #6b7280; /* Grey color for description */
  line-height: 1.5; /* Improved readability through line spacing */
}

.home-link {
  display: inline-block;
  margin-top: 20px;
  padding: 12px 24px;
  font-size: 1rem;
  font-weight: 600;
  color: #ffffff;
  background: linear-gradient(90deg, #6366f1, #38bdf8); /* Gradient background */
  border-radius: 50px;
  text-decoration: none;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.home-link:hover {
  background: linear-gradient(90deg, #38bdf8, #6366f1); /* Reversed gradient on hover */
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.25);
}

/* Animations */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes float {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

/* Mobile Optimization */
@media (max-width: 480px) {
  .not-found{
    margin-top: -25%; /* Added margin-top for mobile */
  }
  .title {
    font-size: 5rem; /* Adjust title size for smaller screens */
  }

  .description {
    font-size: 1rem; /* Ensure readability on mobile */
  }

  .home-link {
    width: 100%;
    max-width: 300px;
    padding: 1em; /* Easier tap interaction for mobile users */
    font-size: 1.1rem; /* Slightly larger text for better readability */
  }
}
  </style>