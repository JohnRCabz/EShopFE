<template>
  <div
    class="p-6 max-w-5xl mx-auto bg-gray-100 rounded-xl shadow-md space-y-4 mt-20"
  >
    <div class="flex justify-between items-center">
      <div class="flex items-center space-x-2">
        <i class="fas fa-shopping-cart text-2xl"></i>
        <h1 class="text-2xl font-bold">E-Shop</h1>
      </div>
      <button
        @click="showModal = true"
        class="bg-blue-500 text-white px-4 py-2 rounded"
      >
        Add New Item
      </button>
    </div>
    <div class="flex space-x-4">
      <!-- Left Column: Item List -->
      <div class="w-1/2">
        <div class="bg-white p-4 rounded-lg shadow-md mb-4">
          <h2 class="text-xl font-semibold mb-4">Items</h2>
          <ul class="space-y-2">
            <li
              v-if="items.length === 0"
              class="text-center text-green-500 font-bold"
            >
              No items available.
            </li>
            <li
              v-for="(item, index) in items"
              :key="index"
              class="p-4 rounded-lg shadow-md hover:shadow-lg transition-shadow duration-300 border-t-4"
              :class="{
                'border-red-500': item.quantity < 10,
                'border-gray-300': item.quantity >= 10,
              }"
            >
              <div class="flex justify-between items-center">
                <span class="font-semibold text-lg">
                  {{ item.name }}
                  <i
                    v-if="item.isHot"
                    class="fas fa-fire text-red-500 ml-2"
                    title="This item is always being bought by the customer."
                  ></i>
                </span>
                <div class="flex space-x-2 items-center">
                  <input
                    type="number"
                    v-model.number="item.addQuantity"
                    min="1"
                    :max="item.quantity"
                    class="w-16 p-1 border rounded"
                    placeholder="Qty"
                  />
                  <button
                    @click="addToCart(item)"
                    class="bg-green-500 text-white px-2 py-1 rounded hover:bg-green-600 transition-colors duration-300"
                  >
                    Add to Cart
                  </button>
                  <button
                    v-if="item.quantity < 10"
                    class="bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600 transition-colors duration-300"
                    title="Refill stock"
                  >
                    <i class="fas fa-exclamation-circle"></i>
                  </button>
                </div>
              </div>
              <hr class="w-full my-2 border-t border-gray-300" />
              <div class="text-left w-full">
                <p class="text-sm text-gray-600">
                  <strong>Description:</strong> {{ item.description }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Price:</strong> ${{ item.price }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Quantity:</strong> {{ item.quantity }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Category:</strong> {{ item.category }}
                </p>
              </div>
            </li>
          </ul>
        </div>
      </div>
      <!-- Right Column: Cart List -->
      <div class="w-1/2">
        <div class="bg-white p-4 rounded-lg shadow-md">
          <div class="flex justify-between items-center mb-4">
            <h2 class="text-xl font-semibold">Cart</h2>
            <div class="flex items-center space-x-2">
              <label for="budget" class="font-bold text-sm">Budget</label>
              <input
                id="budget"
                v-model.number="budget"
                type="number"
                placeholder="Enter your budget"
                class="p-2 border rounded w-24"
              />
            </div>
          </div>
          <ul class="space-y-2">
            <li
              v-if="cartItems.items.length === 0"
              class="text-center text-green-500 font-bold"
            >
              Your cart is empty.
            </li>
            <li
              v-for="(cartItem, index) in cartItems.items"
              :key="index"
              class="p-4 rounded-lg shadow-md hover:shadow-lg transition-shadow duration-300 border-t-4 border-gray-300"
            >
              <div class="flex justify-between items-center">
                <span class="font-semibold text-lg">{{
                  cartItem.item.name
                }}</span>
                <div class="flex space-x-2">
                  <button
                    @click="removeFromCart(index)"
                    class="bg-red-500 text-white px-2 py-1 rounded hover:bg-red-600 transition-colors duration-300"
                  >
                    <i class="fas fa-trash-alt"></i>
                  </button>
                </div>
              </div>
              <hr class="w-full my-2 border-t border-gray-300" />
              <div class="text-left w-full">
                <p class="text-sm text-gray-600">
                  <strong>Description:</strong> {{ cartItem.item.description }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Price:</strong> ${{ cartItem.item.price }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Quantity:</strong> {{ cartItem.quantity }}
                </p>
                <p class="text-sm font-medium">
                  <strong>Category:</strong> {{ cartItem.item.category }}
                </p>
              </div>
            </li>
          </ul>
          <div class="mt-4">
            <p class="text-lg font-semibold">
              Total Amount: ${{ cartItems.total_amount }}
            </p>
            <p class="text-lg font-semibold">
              Total Quantity: {{ cartItems.total_quantity }}
            </p>
            <button
              @click="checkout"
              class="bg-blue-500 text-white px-4 py-2 rounded mt-4"
            >
              Checkout
            </button>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal for Adding New Item -->
    <div
      v-if="showModal"
      class="fixed inset-0 bg-gray-600 bg-opacity-50 flex items-center justify-center"
    >
      <div class="bg-white p-6 rounded-lg shadow-md w-full max-w-md">
        <h2 class="text-xl font-semibold mb-4">Add New Item</h2>
        <form @submit.prevent="addItem" class="flex flex-col space-y-2">
          <label for="item-name" class="font-bold text-sm text-left"
            >Item Name</label
          >
          <input
            id="item-name"
            v-model="newItemName"
            placeholder="Add a new item name"
            class="p-2 border rounded"
            :class="{ 'border-red-500': nameError }"
          />
          <span v-if="nameError" class="text-red-500 text-sm">{{
            nameError
          }}</span>

          <label for="item-description" class="font-bold text-sm text-left"
            >Item Description</label
          >
          <input
            id="item-description"
            v-model="newItemDescription"
            placeholder="Add a new item description"
            class="p-2 border rounded"
            :class="{ 'border-red-500': descriptionError }"
          />
          <span v-if="descriptionError" class="text-red-500 text-sm">{{
            descriptionError
          }}</span>

          <label for="item-price" class="font-bold text-sm text-left"
            >Item Price</label
          >
          <input
            id="item-price"
            v-model="newItemPrice"
            type="number"
            placeholder="Add a new item price"
            class="p-2 border rounded"
            :class="{ 'border-red-500': priceError }"
          />
          <span v-if="priceError" class="text-red-500 text-sm">{{
            priceError
          }}</span>

          <label for="item-quantity" class="font-bold text-sm text-left"
            >Item Quantity</label
          >
          <input
            id="item-quantity"
            v-model="newItemquantity"
            type="number"
            placeholder="Add a new item quantity"
            class="p-2 border rounded"
            :class="{ 'border-red-500': quantityError }"
          />
          <span v-if="quantityError" class="text-red-500 text-sm">{{
            quantityError
          }}</span>

          <label for="item-category" class="font-bold text-sm text-left"
            >Item Category</label
          >
          <input
            id="item-category"
            v-model="newItemCategory"
            placeholder="Add a new item category"
            class="p-2 border rounded"
            :class="{ 'border-red-500': categoryError }"
          />
          <span v-if="categoryError" class="text-red-500 text-sm">{{
            categoryError
          }}</span>

          <div class="flex justify-end space-x-2">
            <button
              type="button"
              @click="showModal = false"
              class="bg-gray-500 text-white px-4 py-2 rounded"
            >
              Cancel
            </button>
            <button
              type="submit"
              class="bg-blue-500 text-white px-4 py-2 rounded"
            >
              Add Item
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import axiosInstance from "../axiosInstance";

interface Item {
  id?: number;
  name: string;
  description: string;
  price: number;
  quantity: number;
  category: string;
  addQuantity?: number;
  isHot?: boolean;
  created_at: string;
}

interface CartItem {
  id: number;
  item_id: number;
  quantity: number;
  item: Item;
}

interface Cart {
  items: CartItem[];
  total_amount: number;
  total_quantity: number;
}

export default defineComponent({
  name: "EshopDashboard",
  data() {
    return {
      showModal: false,
      newItemName: "",
      newItemDescription: "",
      newItemPrice: 0,
      newItemquantity: 0,
      newItemCategory: "",
      items: [] as Item[],
      cartItems: {
        items: [] as CartItem[],
        total_amount: 0,
        total_quantity: 0,
      } as Cart,
      budget: 0,
      nameError: "",
      descriptionError: "",
      priceError: "",
      quantityError: "",
      categoryError: "",
    };
  },
  async created() {
    try {
      const response = await axiosInstance.get("/items");
      if (response.data) {
        this.items = response.data;
      }
    } catch (error) {
      console.error("Error fetching items:", error);
    }

    try {
      const response = await axiosInstance.get("/cart");
      if (response.data) {
        this.cartItems = response.data;
      }
    } catch (error) {
      console.error("Error fetching cart items:", error);
    }
  },
  watch: {
    items: {
      handler(newItems) {
        const today = new Date();
        newItems.forEach(
          (item: {
            created_at: string | number | Date;
            quantity: number;
            isHot: boolean;
          }) => {
            const createdAt = new Date(item.created_at);
            const daysSinceCreation =
              (today.getTime() - createdAt.getTime()) / (1000 * 3600 * 24);
            if (item.quantity < 5 && daysSinceCreation < 3) {
              item.isHot = true;
            } else {
              item.isHot = false;
            }
          }
        );
      },
      deep: true,
    },
  },
  methods: {
    async addItem() {
      this.nameError = "";
      this.descriptionError = "";
      this.priceError = "";
      this.quantityError = "";
      this.categoryError = "";

      if (this.newItemName.trim() === "") {
        this.nameError = "Name is required.";
      }
      if (this.newItemDescription.trim() === "") {
        this.descriptionError = "Description is required.";
      }
      if (this.newItemPrice <= 0) {
        this.priceError = "Price must be greater than zero.";
      }
      if (this.newItemquantity <= 0) {
        this.quantityError = "Quantity must be greater than zero.";
      }
      if (this.newItemCategory.trim() === "") {
        this.categoryError = "Category is required.";
      }

      if (
        !this.nameError &&
        !this.descriptionError &&
        !this.priceError &&
        !this.quantityError &&
        !this.categoryError
      ) {
        const item = {
          name: this.newItemName.trim(),
          description: this.newItemDescription.trim(),
          price: this.newItemPrice,
          quantity: this.newItemquantity,
          category: this.newItemCategory.trim(),
          created_at: new Date().toISOString(),
        };
        try {
          const response = await axiosInstance.post("/items", item);
          if (response.data) {
            this.items.push(response.data);
          }
          this.newItemName = "";
          this.newItemDescription = "";
          this.newItemPrice = 0;
          this.newItemquantity = 0;
          this.newItemCategory = "";
          this.showModal = false;
        } catch (error) {
          console.error("Error adding item:", error);
        }
      }
    },
    async addToCart(item: Item) {
      if (
        !item.addQuantity ||
        item.addQuantity <= 0 ||
        item.addQuantity > item.quantity
      ) {
        alert("Please enter a valid quantity.");
        return;
      }
      try {
        await axiosInstance.post("/cart", {
          item_id: item.id,
          quantity: item.addQuantity,
        });
        const response = await axiosInstance.get("/cart");
        if (response.data) {
          this.cartItems = response.data;
        }
        item.quantity -= item.addQuantity;
        item.addQuantity = 0;
      } catch (error) {
        console.error("Error adding item to cart:", error);
      }
    },
    async removeFromCart(index: number) {
      const cartItem = this.cartItems.items[index];
      try {
        await axiosInstance.delete(`/cart/${cartItem.id}`);
        const response = await axiosInstance.get("/cart");
        if (response.data) {
          this.cartItems = response.data;
        }
        const itemsResponse = await axiosInstance.get("/items");
        if (itemsResponse.data) {
          this.items = itemsResponse.data;
        }
      } catch (error) {
        console.error("Error removing item from cart:", error);
      }
    },
    async checkout() {
      if (this.budget > 0 && this.cartItems.total_amount > this.budget) {
        alert("You have exceeded your budget!");
        return;
      }
      try {
        await axiosInstance.post("/cart/checkout");
        const response = await axiosInstance.get("/cart");
        if (response.data) {
          this.cartItems = response.data;
        }
        alert("Checkout successful!");
      } catch (error) {
        console.error("Error during checkout:", error);
        alert("Checkout failed. Please try again.");
      }
    },
  },
});
</script>

<style scoped>
.border-red-500 {
  border-color: #f56565;
}
</style>
