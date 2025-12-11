<template>
  <section>
    <div
      class="d-flex justify-content-between align-items-end mb-4 flex-wrap gap-3"
    >
      <div>
        <p class="section-kicker text-uppercase mb-1">Explore reviews</p>
        <h1 class="page-title mb-0">All Reviews</h1>
        <p class="text-muted mt-1 small">
          Movies, books, games &amp; music – curated by the team for you.
        </p>
      </div>

      <div
        v-if="reviews.length"
        class="text-muted small fw-medium text-end reviews-count">
        Showing
        <span class="text-primary fw-semibold">{{ filteredReviews.length }}</span>
        of
        <span class="fw-semibold">{{ reviews.length }}</span>
        reviews
      </div>
    </div>

    <div v-if="categories.length" class="mb-3 d-flex flex-wrap gap-2">
      <button
        v-for="cat in categories"
        :key="cat"
        type="button"
        class="btn btn-sm filter-chip"
        :class="{ 'filter-chip--active': selectedCategory === cat }"
        @click="selectedCategory = cat"
      >
        {{ cat }}
      </button>
    </div>

    <div class="mb-4">
      <label for="search" class="form-label mb-2 fw-medium">Search reviews</label>
      <div class="input-group search-input-group">
        <span class="input-group-text">🔍</span>
        <input
          id="search"
          v-model="searchQuery"
          type="text"
          class="form-control"
          placeholder="Search by title, category, or reviewer..."
        />
      </div>
    </div>

    <div v-if="isLoading" class="alert alert-info soft-alert" role="alert">
      Loading reviews...
    </div>

    <div v-else-if="error" class="alert alert-danger soft-alert" role="alert">
      {{ error }}
    </div>

    <div v-else>
      <div v-if="filteredReviews.length === 0" class="alert alert-secondary soft-alert">
        No reviews found for "{{ searchQuery }}".
      </div>

      <div class="row g-4">
        <div
          v-for="review in filteredReviews"
          :key="review.id"
          class="col-12 col-md-6 col-xl-4"
          @click="$router.push(`/reviews/${review.slug}`)"
          style="cursor: pointer;"
        >
          <article class="card card-review h-100 border-0">
            <div class="card-img-container position-relative overflow-hidden">
              <img
                v-if="getCoverUrl(review)"
                :src="getCoverUrl(review)"
                class="card-img-top review-cover-img"
                :alt="review.title"
                loading="lazy"
              />
              <span class="badge-category-img">
                {{ review.category }}
              </span>
              <span class="rating-badge-img">
                ★ {{ review.rating }}/10
              </span>
            </div>

            <div class="card-body d-flex flex-column">
              <h2 class="card-title h5 mb-2 text-title">
                {{ review.title }}
              </h2>

              <p class="card-text review-summary mb-3">
                {{ review.summary }}
              </p>

              <div class="mt-auto card-footer">
                <span class="card-reviewer">👤 {{ review.reviewer }}</span>
                <span class="card-date">📅 {{ formatDate(review.date) }}</span>
              </div>
            </div>
          </article>
        </div>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const API_ORIGIN = 'https://review-site-backend-uw6k.onrender.com'
const API_BASE = `${API_ORIGIN}/api`

const reviews = ref([])
const searchQuery = ref('')
const selectedCategory = ref('All')
const isLoading = ref(true)
const error = ref(null)

const fetchReviews = async () => {
  const url = `${API_BASE}/reviews?populate=*`

  try {
    const res = await fetch(url)
    if (!res.ok) {
      throw new Error(`Request failed with status ${res.status}`)
    }

    const data = await res.json()
    reviews.value = data.data || []
  } catch (err) {
    console.error('[HomeView] Fetch error:', err)
    error.value = 'Failed to load reviews.'
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchReviews)

const categories = computed(() => {
  const set = new Set(reviews.value.map((r) => r.category))
  return ['All', ...Array.from(set)]
})

const filteredReviews = computed(() => {
  let list = reviews.value

  if (selectedCategory.value !== 'All') {
    list = list.filter((r) => r.category === selectedCategory.value)
  }

  if (!searchQuery.value.trim()) return list

  const q = searchQuery.value.toLowerCase()
  return list.filter((r) => {
    return (
      r.title.toLowerCase().includes(q) ||
      r.category.toLowerCase().includes(q) ||
      (r.reviewer && r.reviewer.toLowerCase().includes(q))
    )
  })
})

const formatDate = (value) => {
  if (!value) return ''
  const d = new Date(value)
  if (Number.isNaN(d.getTime())) return ''
  return d.toLocaleDateString()
}

const getCoverUrl = (review) => {
 return review.coverImage?.url || null;
}
</script>

<style scoped>
.card-review {
  border-radius: var(--radius-lg);
  background-color: var(--card-bg);
  border: 1px solid var(--card-border);
  transition: transform 0.3s ease, box-shadow 0.3s ease;
}

.card-review:hover {
  transform: translateY(-10px);
  box-shadow: 0 15px 30px rgba(15, 23, 42, 0.15);
  border-color: rgba(79, 70, 229, 0.3);
  background-color: #ffffff;
}

.card-img-container {
  position: relative;
  width: 100%;
  height: 220px;
  overflow: hidden;
  background-color: #f8f9fa;
  border-top-left-radius: var(--radius-lg);
  border-top-right-radius: var(--radius-lg);
}

.review-cover-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
  transition: transform 0.5s ease;
}

.card-review:hover .review-cover-img {
  transform: scale(1.1);
}

.badge-category-img {
  position: absolute;
  top: 15px;
  left: 15px;
  background: var(--primary);
  color: white;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-size: 0.8rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
  box-shadow: 0 3px 10px rgba(79, 70, 229, 0.3);
  z-index: 1;
}

.rating-badge-img {
  position: absolute;
  bottom: 15px;
  right: 15px;
  background: rgba(0, 0, 0, 0.8);
  color: white;
  padding: 0.3rem 0.8rem;
  border-radius: 15px;
  font-weight: bold;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.2rem;
  z-index: 1;
}

.card-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding-top: 1rem;
  border-top: 1px solid var(--card-border);
  font-size: 0.9rem;
  color: var(--text-muted);
  margin-top: auto;
}

.card-reviewer, .card-date {
  display: flex;
  align-items: center;
  gap: 0.3rem;
}

@media (max-width: 768px) {
  .card-img-container {
    height: 180px;
  }
}
</style>