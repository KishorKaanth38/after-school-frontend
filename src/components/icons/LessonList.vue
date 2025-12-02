<template>
  <section class="lessons-page">
    
    <!-- LESSON PAGE -->
    <div v-if="currentPage === 'lessons'">

      <!-- HEADER SECTION -->
      <header class="lessons-header">
        <h2>Available Lessons</h2>
        <p>Browse after-school classes and add them to your cart.</p>

        <input
          v-model="searchQuery"
          type="text"
          class="search-bar"
          placeholder="Search lessons..."
        />
      </header>

      <!-- SORT SECTION -->
      <div class="sort-row">
        <div class="sort-group">
          <label class="sort-label">Sort by:</label>
          <select v-model="sortKey" class="form-select sort-select">
            <option value="subject">Subject</option>
            <option value="location">Location</option>
            <option value="price">Price</option>
            <option value="spaces">Spaces</option>
          </select>
        </div>

        <button class="btn btn-outline-primary btn-sm" @click="toggleSortOrder">
          Sort: {{ sortOrder === 1 ? 'Ascending' : 'Descending' }}
        </button>
      </div>

      <!-- LESSON GRID -->
      <div class="lesson-grid">
        <article
          v-for="lesson in sortedLessons"
          :key="lesson._id"
          class="lesson-card shadow-sm"
        >
          <div v-if="lesson.image" class="lesson-cover">
            <img :src="getImage(lesson.image)" class="cover-img" />
          </div>

          <h5 class="lesson-title">{{ lesson.subject }}</h5>

          <p class="lesson-location">
            <i class="bi bi-geo-alt-fill"></i> {{ lesson.location }}
          </p>

          <p class="lesson-price">£{{ lesson.price }}</p>
          <p class="lesson-spaces">Spaces left: {{ lesson.spaces }}</p>

          <button
            class="btn btn-primary w-100 mt-2"
            :disabled="lesson.spaces === 0"
            @click="addToCart(lesson)"
          >
            {{ lesson.spaces === 0 ? "Full" : "Add to Cart" }}
          </button>
        </article>
      </div>
    </div>

    <!-- CART PAGE -->
    <div
      v-if="currentPage === 'cart'"
      class="cart-page shadow-sm bg-white rounded"
    >
      <h2 class="cart-title">🛒 Your Cart</h2>

      <div v-if="cart.length === 0" class="cart-empty">Your cart is empty.</div>

      <div v-else>
        <ul class="list-group mb-3">
          <li
            v-for="item in cart"
            :key="item._id"
            class="list-group-item d-flex justify-content-between align-items-center"
          >
            <div>
              <strong>{{ item.subject }}</strong>
              <small class="text-muted d-block">
                £{{ item.price }} × {{ item.quantity }}
              </small>
            </div>

            <button
              class="btn btn-sm btn-outline-danger"
              @click="removeFromCart(item)"
            >
              Remove
            </button>
          </li>
        </ul>

        <h4 class="text-end fw-bold">Total: £{{ totalPrice }}</h4>

        <!-- Checkout -->
        <div class="checkout-form">
          <div class="mb-2">
            <label class="form-label">Name</label>
            <input v-model="customerName" type="text" class="form-control" />
            <p class="text-danger" v-if="nameError">{{ nameError }}</p>
          </div>

          <div class="mb-3">
            <label class="form-label">Phone</label>
            <input v-model="customerPhone" type="text" class="form-control" />
            <p class="text-danger" v-if="phoneError">{{ phoneError }}</p>
          </div>

          <button
            class="btn btn-success w-100 fw-semibold"
            @click="checkout"
          >
            Confirm Order
          </button>
        </div>
      </div>
    </div>

    <!-- SUCCESS POPUP -->
    <div v-if="showPopup" class="popup">
      ✅ Order placed successfully!
    </div>

    <!-- ERROR POPUP -->
    <div v-if="errorPopup" class="popup error-popup">
      ❌ {{ errorPopup }}
    </div>

  </section>
</template>

<script>
export default {
  props: ["currentPage"],

  data() {
    return {
      lessons: [],
      cart: [],
      sortKey: "subject",
      sortOrder: 1,
      searchQuery: "",
      customerName: "",
      customerPhone: "",
      nameError: "",
      phoneError: "",
      showPopup: false,
      errorPopup: "",
    };
  },

  mounted() {
    this.fetchLessons();
  },

  computed: {
    sortedLessons() {
      const q = this.searchQuery.toLowerCase();

      return [...this.lessons]
        .filter(
          (lesson) =>
            lesson.subject.toLowerCase().includes(q) ||
            lesson.location.toLowerCase().includes(q) ||
            lesson.price.toString().includes(q) ||
            lesson.spaces.toString().includes(q)
        )
        .sort((a, b) => {
          let A = a[this.sortKey];
          let B = b[this.sortKey];
          if (typeof A === "string") A = A.toLowerCase();
          if (typeof B === "string") B = B.toLowerCase();
          return A < B ? -1 * this.sortOrder : A > B ? 1 * this.sortOrder : 0;
        });
    },

    totalPrice() {
      return this.cart.reduce((sum, i) => sum + i.price * i.quantity, 0);
    },

    isFormValid() {
      return (
        this.customerName.trim().length > 0 &&
        /^[0-9]+$/.test(this.customerPhone) &&
        this.cart.length > 0
      );
    },
  },

  methods: {
    getImage(filename) {
      return import.meta.env.BASE_URL + "images/" + filename;
    },

    async fetchLessons() {
      const res = await fetch(
        "https://after-school-backend-tuu4.onrender.com/lessons"
      );
      const data = await res.json();

      const map = {
        Art: "ART.jpg",
        Drama: "DRAMA.jpg",
        English: "ENGLISH.jpg",
        Geography: "GEOGRAPHY.png",
        History: "HISTORY.png",
        Math: "MATHS.jpg",
        Music: "MUSIC.jpg",
        Programming: "PROGRAMMING.jpg",
        Science: "SCIENCE.jpg",
        Sports: "SPORTS.jpg",
      };

      this.lessons = data.map((l) => ({
        ...l,
        image: map[l.subject] || null,
      }));
    },

    toggleSortOrder() {
      this.sortOrder *= -1;
    },

    addToCart(lesson) {
      const item = this.cart.find((i) => i._id === lesson._id);
      item ? item.quantity++ : this.cart.push({ ...lesson, quantity: 1 });

      if (lesson.spaces > 0) lesson.spaces--;

      this.$emit("update-cart", this.cart.length);
    },

    removeFromCart(item) {
      const lesson = this.lessons.find((l) => l._id === item._id);
      if (lesson) lesson.spaces += item.quantity;

      this.cart = this.cart.filter((i) => i._id !== item._id);
      this.$emit("update-cart", this.cart.length);
    },

    clearErrorPopup() {
      setTimeout(() => (this.errorPopup = ""), 2000);
    },

    validateCheckoutForm() {
      this.nameError = "";
      this.phoneError = "";
      this.errorPopup = "";

      if (!this.customerName.trim()) {
        this.nameError = "Name is required.";
        this.errorPopup = "Please enter your name.";
        this.clearErrorPopup();
        return false;
      }

      if (!this.customerPhone.trim()) {
        this.phoneError = "Phone number is required.";
        this.errorPopup = "Please enter a phone number.";
        this.clearErrorPopup();
        return false;
      }

      if (!/^\d+$/.test(this.customerPhone)) {
        this.phoneError = "Phone number must contain only digits.";
        this.errorPopup = "Phone number must contain only digits.";
        this.clearErrorPopup();
        return false;
      }

      if (this.customerPhone.length < 7 || this.customerPhone.length > 15) {
        this.phoneError = "Phone number length is invalid.";
        this.errorPopup = "Phone number length is invalid.";
        this.clearErrorPopup();
        return false;
      }

      return true;
    },

    async checkout() {
      if (!this.validateCheckoutForm()) return;

      try {
        const res = await fetch(
          "https://after-school-backend-tuu4.onrender.com/orders",
          {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({
              name: this.customerName,
              phone: this.customerPhone,
              cart: this.cart.map((i) => ({
                _id: i._id,
                quantity: i.quantity,
              })),
            }),
          }
        );

        const data = await res.json();

        if (!res.ok) {
          this.errorPopup = data.error || "Failed to place order.";
          this.clearErrorPopup();
          return;
        }

        this.showPopup = true;
        setTimeout(() => (this.showPopup = false), 2500);

        this.cart = [];
        this.customerName = "";
        this.customerPhone = "";

        await this.fetchLessons();
        this.$emit("update-cart", 0);

      } catch (err) {
        console.error("Checkout failed:", err);
        this.errorPopup = "Failed to place order!";
        this.clearErrorPopup();
      }
    },
  },
};
</script>

<style scoped>
/* Header */
.lessons-header {
  margin-bottom: 2rem;
}

.search-bar {
  width: 280px;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #d0d4da;
  margin-top: 1rem;
}

/* Sort row */
.sort-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

/* Grid */
.lesson-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 1.5rem;
}

/* Cover image */
.lesson-cover {
  width: 100%;
  height: 110px;
  border-radius: 10px;
  overflow: hidden;
  margin-bottom: 0.75rem;
  background: #f0f0f0;
}

.cover-img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Cards */
.lesson-card {
  background: white;
  border-radius: 12px;
  border: 1px solid #e5e7eb;
  padding: 1.25rem;
  transition: 0.18s ease;
}

.lesson-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 10px 24px rgba(0, 0, 0, 0.08);
}

/* Popup */
.popup {
  position: fixed;
  top: 20px;
  right: 20px;
  background: #28a745;
  color: #fff;
  padding: 12px 20px;
  border-radius: 8px;
}

.error-popup {
  background: #dc3545;
}
</style>
