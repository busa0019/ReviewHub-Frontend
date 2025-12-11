<template>
  <div class="review-detail-container">
    <button @click="$router.back()" class="back-btn">← Back to Reviews</button>
    
    <div v-if="isLoading" class="loading">
      <div class="spinner"></div>
      Loading review...
    </div>
    
    <div v-else-if="error" class="error alert alert-danger">
      {{ error }}
    </div>
    
    <div v-else-if="review" class="review-content">
      <article class="review-article">
        <header class="review-header">
          <div class="review-meta">
            <span class="review-category">{{ review.category }}</span>
            <span class="review-rating">{{ review.rating }}/10</span>
          </div>
          <h1>{{ review.title }}</h1>
          <div class="review-info">
            <span class="reviewer">👤 By {{ review.reviewer }}</span>
            <span class="date">📅 {{ formatDate(review.date) }}</span>
          </div>
        </header>
        
        <div class="review-image" v-if="coverUrl">
          <img 
            :src="coverUrl" 
            :alt="review.title"
            width="600"
          />
        </div>
        
        <div class="review-body">
          <div class="summary-section">
            <h2>Summary</h2>
            <p>{{ review.summary }}</p>
          </div>
          
          <div class="full-review">
            <h2>Full Review</h2>
            <div class="review-content" v-html="review.content"></div>
          </div>
        </div>
        
        <footer class="review-footer">
          <button @click="$router.push('/')" class="back-home-btn">← All Reviews</button>
          <div class="footer-note">
            Thanks for reading! Check out more reviews for honest opinions.
          </div>
        </footer>
      </article>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const route = useRoute()
const router = useRouter()

const API_ORIGIN = 'https://review-site-backend-uw6k.onrender.com'
const API_BASE = `${API_ORIGIN}/api`

const review = ref(null)
const isLoading = ref(true)
const error = ref(null)

const fetchReview = async () => {
  try {
    const slug = route.params.slug
    const res = await fetch(
      `${API_BASE}/reviews?filters[slug][$eq]=${slug}&populate=*`
    )
    if (!res.ok) throw new Error('Failed to fetch review')
    const data = await res.json()
    review.value = data.data[0] || null
  } catch (err) {
    console.error('[Detail] Fetch error:', err)
    error.value = 'Failed to load review.'
  } finally {
    isLoading.value = false
  }
}

const coverUrl = computed(() => {
  const coverImage = review.value?.coverImage;
  if (!coverImage) return null;
  
  return coverImage.formats?.large?.url || 
         coverImage.formats?.medium?.url || 
         coverImage.url || 
         null;
})

const formatDate = (value) => {
  if (!value) return ''
  const d = new Date(value)
  if (Number.isNaN(d.getTime())) return ''
  return d.toLocaleDateString('en-US', {
    year: 'numeric',
    month: 'long',
    day: 'numeric'
  })
}

onMounted(() => {
  fetchReview()
})
</script>

<style scoped>
.review-detail-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 0 1rem 3rem;
}

.back-btn {
  background: none;
  border: none;
  color: var(--primary);
  font-size: 1rem;
  cursor: pointer;
  padding: 0.8rem 1rem;
  margin-bottom: 2rem;
  border-radius: 5px;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  transition: all 0.3s ease;
  font-weight: 600;
}

.back-btn:hover {
  background: rgba(79, 70, 229, 0.1);
  transform: translateX(-5px);
  padding-left: 1.2rem;
}

.loading, .error {
  text-align: center;
  padding: 3rem;
  color: var(--text-muted);
  background: white;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
}

.spinner {
  width: 50px;
  height: 50px;
  border: 3px solid #f3f3f3;
  border-top: 3px solid var(--primary);
  border-radius: 50%;
  margin: 0 auto 1rem;
  animation: spin 1s linear infinite;
}

.review-article {
  background: white;
  border-radius: 10px;
  overflow: hidden;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.1);
  animation: fadeIn 0.5s ease;
}

.review-header {
  padding: 2.5rem 2.5rem 1.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-bottom: 1px solid #eee;
}

.review-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.5rem;
}

.review-category {
  background: linear-gradient(45deg, #4338ca, #4f46e5);
  color: white;
  padding: 0.5rem 1.2rem;
  border-radius: 20px;
  font-weight: 600;
  font-size: 0.9rem;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  box-shadow: 0 3px 10px rgba(79, 70, 229, 0.3);
}

.review-rating {
  background: linear-gradient(45deg, #f59e0b, #fbbf24);
  color: white;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  font-weight: bold;
  font-size: 1.1rem;
  box-shadow: 0 3px 10px rgba(245, 158, 11, 0.3);
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

.review-rating::before {
  content: '★';
  font-size: 1rem;
}

.review-article h1 {
  color: var(--text-main);
  font-size: 2.2rem;
  line-height: 1.2;
  margin: 1rem 0;
  position: relative;
  padding-bottom: 1rem;
}

.review-article h1::after {
  content: '';
  position: absolute;
  bottom: 0;
  left: 0;
  width: 100px;
  height: 3px;
  background: linear-gradient(to right, var(--primary), var(--accent));
  border-radius: 2px;
}

.review-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: var(--text-muted);
  font-size: 0.9rem;
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid #eee;
}

.reviewer, .date {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.reviewer {
  font-weight: 600;
  color: var(--primary);
}

.review-image {
  width: 100%;
  padding: 2rem;
  text-align: center;
  background: #f8f9fa;
  border-bottom: 1px solid #eee;
}

.review-image img {
  max-width: 100%;
  height: auto;
  border-radius: 10px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.2);
  transition: transform 0.5s ease;
}

.review-image img:hover {
  transform: scale(1.02);
}

.review-body {
  padding: 2.5rem;
}

.review-body h2 {
  color: var(--text-main);
  margin: 2rem 0 1rem 0;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid var(--primary);
  font-size: 1.5rem;
}

.summary-section {
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  padding: 1.5rem;
  border-radius: 10px;
  margin-bottom: 2rem;
  border-left: 4px solid var(--primary);
}

.summary-section p {
  font-size: 1.1rem;
  line-height: 1.7;
  color: var(--text-main);
  font-style: italic;
}

.full-review {
  background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
  padding: 1.5rem;
  border-radius: 10px;
  margin-top: 2rem;
  border-left: 4px solid var(--accent);
}

.full-review h2 {
  color: var(--text-main);
  margin-top: 0;
  margin-bottom: 1rem;
  padding-bottom: 0.5rem;
  border-bottom: 2px solid var(--accent);
  font-size: 1.5rem;
}

.review-content {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 1.05rem;
  line-height: 1.8;
  color: var(--text-main);
}

.review-content :deep(p) {
  margin-bottom: 1.5rem;
  text-align: justify;
}

.review-content :deep(h2),
.review-content :deep(h3),
.review-content :deep(h4) {
  margin-top: 2rem;
  margin-bottom: 1rem;
  color: var(--text-main);
  font-weight: 600;
}

.review-content :deep(h2) {
  font-size: 1.5rem;
  border-bottom: 2px solid var(--primary);
  padding-bottom: 0.5rem;
}

.review-content :deep(h3) {
  font-size: 1.3rem;
}

.review-content :deep(h4) {
  font-size: 1.1rem;
}

.review-content :deep(ul),
.review-content :deep(ol) {
  margin-bottom: 1.5rem;
  padding-left: 1.5rem;
}

.review-content :deep(li) {
  margin-bottom: 0.5rem;
}

.review-footer {
  padding: 2rem 2.5rem;
  background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  border-top: 1px solid #eee;
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1rem;
}

.back-home-btn {
  background: var(--primary);
  color: white;
  border: none;
  padding: 0.8rem 1.5rem;
  border-radius: 25px;
  cursor: pointer;
  font-weight: 600;
  transition: all 0.3s ease;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.back-home-btn:hover {
  background: #4338ca;
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.2);
}

.footer-note {
  color: var(--text-muted);
  font-size: 0.9rem;
  font-style: italic;
  text-align: right;
  flex: 1;
}

@media (max-width: 768px) {
  .review-header {
    padding: 1.5rem 1.5rem 1rem;
  }
  
  .review-meta {
    flex-direction: column;
    gap: 1rem;
    align-items: flex-start;
  }
  
  .review-article h1 {
    font-size: 1.8rem;
  }
  
  .review-info {
    flex-direction: column;
    gap: 0.5rem;
    align-items: flex-start;
  }
  
  .review-body {
    padding: 1.5rem;
  }
  
  .review-footer {
    flex-direction: column;
    text-align: center;
  }
  
  .footer-note {
    text-align: center;
  }
}
</style>