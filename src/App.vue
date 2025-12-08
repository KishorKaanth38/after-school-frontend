<template>
  <div id="app">
    <!-- NAVBAR -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary shadow-sm nav-fixed">
      <div class="container-wide d-flex align-items-center">
        <a class="navbar-brand fw-bold fs-4" href="#">After Classes</a>

        <button
          class="navbar-toggler ms-auto"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#navbarNav"
        >
          <span class="navbar-toggler-icon"></span>
        </button>

        <div class="collapse navbar-collapse" id="navbarNav">
          <ul class="navbar-nav ms-auto align-items-center gap-3">
            <li class="nav-item">
              <a
                class="nav-link"
                :class="{ active: currentPage === 'lessons' }"
                style="cursor:pointer"
                @click="currentPage = 'lessons'"
              >
                Home
              </a>
            </li>

            <li class="nav-item">
              <button
                class="btn btn-warning fw-semibold"
                :disabled="cartCount === 0"
                @click="toggleCartPage"
              >
                🛒 Cart ({{ cartCount }})
              </button>
            </li>
          </ul>
        </div>
      </div>
    </nav>

    <!-- MAIN CONTENT -->
    <main class="page-wrapper">
      <LessonList
        :currentPage="currentPage"
        @update-cart="updateCartCount"
      />
    </main>

    <!-- FOOTER -->
    <footer class="footer text-center">
      © 2025 After Classes | Designed by Kishor Kaanth
    </footer>
  </div>
</template>

<script>
import LessonList from "./LessonList.vue";

export default {
  components: { LessonList },

  data() {
    return {
      currentPage: "lessons",
      cartCount: 0,
    };
  },

  methods: {
    toggleCartPage() {
      if (this.currentPage === "cart") this.currentPage = "lessons";
      else if (this.cartCount > 0) this.currentPage = "cart";
    },

    updateCartCount(count) {
      this.cartCount = count;
    }
  }
};
</script>

<style>

html,
body {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: "Inter", sans-serif;

  /* FULL BACKGROUND IMAGE */
  background-size: cover;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-position: center;
}

/* Apply overlay to entire app including NAVBAR */
#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;

  /* Transparent overlay everywhere */
  background: rgba(255, 255, 255, 0.75);
  backdrop-filter: blur(3px);
}

/* Navbar visible + floating on top */
.navbar {
  position: sticky;
  top: 0;
  z-index: 999;
  background: rgba(13, 110, 253, 0.95) !important; /* Strong Blue */
  backdrop-filter: blur(4px);
}

/* Center content */
.container-wide {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}

/* Ensure the top content is not hidden behind Nav */
.page-wrapper {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto 3rem auto;

  padding: 5rem 1.5rem 2rem; 
  /* TOP padding increased because navbar height ~60px */
  flex: 1;
}

.footer {
  background: #212529;
  color: white;
  padding: 0.75rem;
  font-size: 0.85rem;
}

/* Hover effects */
.nav-link:hover {
  color: #ffe082 !important;
}

.navbar-brand:hover {
  color: #ffeb3b !important;
}

</style>
