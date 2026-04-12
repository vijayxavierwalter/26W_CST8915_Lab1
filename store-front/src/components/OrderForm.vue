<template>
  <div class="store-container">
    <!-- Header -->
    <header class="store-header">
      <h1>Algonquin Pet Store</h1>
      <p class="tagline">Meals for Pets, Lessons for Students!</p>
    </header>

    <!-- Animal Images -->
    <div class="animal-images">
      <img src="pictures/dog.jpg" alt="Dog" />
      <img src="pictures/cat.jpg" alt="Cat" />
      <img src="pictures/bird.jpg" alt="Bird" />
    </div>

    <!-- Product Selection -->
    <div v-if="products.length">
      <h2>Select a Product</h2>

      <div
        v-for="product in products"
        :key="product.id"
        class="product-item"
      >
        <input
          type="radio"
          :id="product.id"
          v-model="selectedProduct"
          :value="product"
        />
        <label :for="product.id">
          <strong>{{ product.name }}</strong> - $
          {{ product.price ? product.price.toFixed(2) : 'N/A' }}
        </label>
      </div>

      <!-- Quantity -->
      <div v-if="selectedProduct" class="quantity-container">
        <label for="quantity">Quantity:</label>
        <input
          type="number"
          v-model="quantity"
          min="1"
          placeholder="Enter quantity"
        />
      </div>

      <!-- Total -->
      <div v-if="selectedProduct" class="total-price">
        <h3>Total Price: ${{ totalPrice.toFixed(2) }}</h3>
      </div>

      <button
        @click="submitOrder"
        :disabled="!selectedProduct || quantity <= 0"
        class="order-button"
      >
        Place Order
      </button>
    </div>

    <div v-else>
      <p>Loading products...</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      products: [],
      selectedProduct: null,
      quantity: 1,
    };
  },

  async created() {
    await this.fetchProducts();
  },

  computed: {
    totalPrice() {
      return this.selectedProduct
        ? this.selectedProduct.price * this.quantity
        : 0;
    },
  },

  methods: {
    // ✅ Fetch products from Product Service
    async fetchProducts() {
      try {
        const response = await fetch(
          "https://product-service-app-cucqgzg2fecegtbq.canadacentral-01.azurewebsites.net/products"
        );

        if (response.ok) {
          this.products = await response.json();
        } else {
          alert("Failed to fetch products.");
        }
      } catch (error) {
        console.error("Error fetching products:", error);
        alert("Failed to fetch products.");
      }
    },

    // ✅ Submit order to Order Service
    async submitOrder() {
      if (!this.selectedProduct || this.quantity <= 0) {
        alert("Please select a product and enter a valid quantity.");
        return;
      }

      try {
        const response = await fetch(
          "https://order-service-app-a9htgzhpbugfe4h5.canadacentral-01.azurewebsites.net/orders",
          {
            method: "POST",
            headers: {
              "Content-Type": "application/json",
            },
            body: JSON.stringify({
              product: this.selectedProduct,
              quantity: this.quantity,
              totalPrice: this.totalPrice,
            }),
          }
        );

        if (!response.ok) {
          throw new Error(`Server error: ${response.status}`);
        }

        alert(
          `Order for ${this.quantity} x ${this.selectedProduct.name} placed successfully! Total: $${this.totalPrice.toFixed(
            2
          )}`
        );
      } catch (error) {
        console.error("Error placing order:", error);
        alert("Failed to place order.");
      }
    },
  },
};
</script>

<style scoped>
.store-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  background-color: rgba(255, 255, 255, 0.9);
  border-radius: 10px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
}

.store-header {
  text-align: center;
  margin-bottom: 20px;
}

.store-header h1 {
  font-size: 2.5rem;
  color: #42b983;
}

.store-header .tagline {
  font-size: 1.2rem;
  color: #777;
  font-style: italic;
}

.animal-images {
  display: flex;
  justify-content: space-around;
  margin-bottom: 20px;
}

.animal-images img {
  width: 200px;
  height: 200px;
  border-radius: 50%;
  object-fit: cover;
}

.product-item {
  display: flex;
  align-items: center;
  margin: 10px 0;
  padding: 10px;
  border: 2px solid #42b983;
  border-radius: 10px;
  background-color: #e3f2fd;
}

.quantity-container {
  margin-top: 15px;
}

.total-price {
  margin-top: 20px;
  text-align: center;
  font-size: 1.3rem;
  color: #ff6f00;
}

.order-button {
  margin-top: 20px;
  padding: 10px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 5px;
  width: 100%;
}

.order-button:disabled {
  background-color: #ccc;
}
</style>
