<template>
  <div id="app">
    <!-- NAVBAR -->
    <nav class="navbar navbar-expand-lg navbar-dark bg-primary shadow-sm">
      <div class="container-wide d-flex align-items-center">
        <a class="navbar-brand fw-bold fs-4" href="#">After-School</a>

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
import LessonList from "./components/icons/LessonList.vue";

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
  background-image: url("/images/BACKGROUND.jpg");
  background-size: cover;
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-position: center;
}

/* white overlay so text is readable */
#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background: rgba(255, 255, 255, 0.65);
  backdrop-filter: blur(2px);
}

.container-wide {
  width: 100%;
  max-width: 1200px;
  margin: 0 auto;
}

.page-wrapper {
  width: 100%;
  max-width: 1200px;
  margin: 2rem auto 3rem auto;
  padding: 0 1.5rem;
  flex: 1;
}

.footer {
  background: #212529;
  color: white;
  padding: 0.75rem;
  font-size: 0.85rem;
}

.nav-link:hover {
  color: #ffe082 !important;
}

.navbar-brand:hover {
  color: #ffeb3b !important;
}
</style>
