<template>
  <div class="app-shell d-flex flex-column min-vh-100">
    <nav class="navbar navbar-expand-lg navbar-dark app-navbar shadow-sm">
      <div class="container">
        <RouterLink class="navbar-brand fw-semibold d-flex align-items-center gap-2" to="/">
          <span class="brand-badge">RH</span>
          <span>Review Hub</span>
        </RouterLink>
        <button
          class="navbar-toggler"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#mainNavbar"
          aria-controls="mainNavbar"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="mainNavbar">
          <ul class="navbar-nav ms-auto mb-2 mb-lg-0 align-items-lg-center gap-lg-2">
            <li class="nav-item">
              <RouterLink class="nav-link" to="/">Home</RouterLink>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <main class="flex-grow-1 py-5">
      <div class="container app-main-container">
        <RouterView />
      </div>
    </main>
    <footer class="app-footer py-3">
      <div class="container text-center small ">
        &copy; {{ year }} Review Hub · Built with Strapi &amp; Vue
      </div>
    </footer>
  </div>
</template>

<script setup>
import { computed } from 'vue'
import { RouterLink, RouterView } from 'vue-router'

const year = computed(() => new Date().getFullYear())
</script>
