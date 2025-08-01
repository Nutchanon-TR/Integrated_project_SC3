<script setup>
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import {
  getSaleItemById,
  deleteSaleItemById,
} from "@/libs/callAPI/apiSaleItem.js";
import { unitPrice, nullCatching } from "@/libs/utils.js";
import { useAlertStore } from "@/stores/alertStore.js";

const route = useRoute();
const router = useRouter();
const product = ref({});
const loading = ref(true);
const quantity = ref(1);
const alertStore = useAlertStore();
const showDeleteModal = ref(false);
const pendingDeleteId = ref(null);

const confirmDeleteProduct = async () => {
  try {
    await deleteSaleItemById(pendingDeleteId.value);
    alertStore.setMessage("The sale item has been deleted.");
    sessionStorage.setItem("item-just-deleted", "true");
    router.push("/sale-items");
  } catch (error) {
    if (error.status === 404) {
      alertStore.setMessage("The requested sale item does not exist.", "error");
      sessionStorage.setItem("item-just-deleted", "true");
      router.push("/sale-items");
    } else {
      alertStore.setMessage("The requested sale item does not exist.", "error");
      sessionStorage.setItem("item-just-deleted", "true");
      router.push("/sale-items");
    }
  } finally {
    showDeleteModal.value = false;
  }
};

const deleteProduct = (id) => {
  pendingDeleteId.value = id;
  showDeleteModal.value = true;
};

onMounted(async () => {
  try {
    loading.value = true;
    const data = await getSaleItemById(route.params.id);
    if (data == undefined) {
      product.value = "404_not_found";
      console.log("product.value: " + product.value);
      setTimeout(() => {
        router.push("/sale-items");
      }, 2000);
    } else {
      product.value = data;
      console.log(product.value);
      console.log(data);
      console.log("product.value: " + product.value.price);
      console.log("product.value: " + typeof product.value.price);
      console.log("product.value: " + product.value.color);
    }
  } catch (error) {
    console.log("โหลดข้อมูลสินค้าไม่สำเร็จ:", error.message);
  } finally {
    loading.value = false;
  }

  if (alertStore.message) {
    setTimeout(() => {
      alertStore.clearMessage();
    }, 3000);
  }
});

// gen มา
const decrementQuantity = () => {
  if (quantity.value > 1) {
    quantity.value--;
  }
};

const incrementQuantity = () => {
  if (quantity.value < product.value.quantity) {
    quantity.value++;
  }
};
</script>

<template>
  <!-- Loading Spinner -->
  <div v-if="loading" class="flex items-center justify-center min-h-[60vh]">
    <div
      class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-blue-600"
    ></div>
  </div>

  <!-- 404 Error Page -->
  <div
    v-else-if="product == '404_not_found'"
    class="flex flex-col items-center justify-center text-center py-20 space-y-8 min-h-[60vh] bg-gray-50"
  >
    <div class="bg-white p-8 rounded-xl shadow-lg max-w-md w-full">
      <!-- 404 Icon -->
      <img
        src="https://static.thenounproject.com/png/4019366-200.png"
        alt="404 Icon"
        class="w-24 h-24 mx-auto opacity-80"
      />

      <!-- Error Message -->
      <!-- <h1 class="itbms-message text-2xl font-bold text-gray-800 mt-6">ไม่พบสินค้าที่ค้นหา</h1> -->
      <p class="itbms-message text-gray-600 mt-2">
        The requested sale item does not exist.
      </p>
    </div>
  </div>

  <!-- Product Detail Page -->
  <div v-else class="itbms-row bg-gray-50 min-h-screen pb-12">
    <!-- Breadcrumb -->
    <div class="bg-white shadow-sm">
      <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-3">
        <div class="flex items-center space-x-2 text-sm text-gray-600">
          <RouterLink
            to="/sale-items"
            class="itbms-home-button hover:text-blue-600 transition"
            >Home</RouterLink
          >
          <span class="text-gray-400">/</span>
          <span class="text-gray-900 font-medium"
            >{{ product.brandName }} {{ product.model }}</span
          >
        </div>
      </div>
    </div>

    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 pt-8">
      <!-- Product Overview Section -->
      <div class="bg-white rounded-xl shadow-md overflow-hidden">
        <div class="md:flex">
          <!-- Product Image Section -->
          <div class="md:w-1/2 p-6 flex items-center justify-center bg-white">
            <div class="relative group">
              <img
                src="https://app-area.riointernational.com.bd/productImages/1738403480BRk6I.png"
                alt="Smartphone Image"
                class="w-full h-80 sm:h-96 object-contain transition duration-300 group-hover:scale-105"
              />
              <div
                class="absolute inset-0 bg-blue-600 opacity-0 group-hover:opacity-5 transition duration-300 rounded-xl"
              ></div>
            </div>
          </div>

          <!-- Product Info Section -->
          <div class="md:w-1/2 p-6 md:p-8 bg-white flex flex-col">
            <!-- Brand & Model -->
            <div class="border-b pb-4">
              <h1 class="itbms-brand text-3xl font-bold text-gray-800">
                {{ product.brandName }}
              </h1>
              <p class="itbms-model text-xl text-gray-700 mt-1">
                {{ product.model }}
              </p>
              <div class="mt-2 flex items-center"></div>
            </div>

            <!-- Price and Stock -->
            <div class="py-4 border-b">
              <div class="flex items-baseline">
                <span class="itbms-price text-3xl font-bold text-blue-600">{{
                  unitPrice(product.price)
                }}</span>
                <span class="itbms-price-unit ml-1 text-lg text-gray-500"
                  >Baht</span
                >
              </div>

              <div class="mt-2 flex items-center space-x-2">
                <span class="text-sm text-gray-500">
                  สินค้าคงเหลือ:
                  <span class="itbms-quantity font-medium">{{
                    product.quantity
                  }}</span>
                  ชิ้น
                </span>
              </div>
            </div>

            <!-- Color -->
            <div class="py-4 border-b">
              <h3 class="text-sm font-medium text-gray-700">
                สี:
                <span class="itbms-color font-semibold">{{
                  nullCatching(product.color)
                }}</span>
              </h3>
              <div class="mt-2 flex items-center space-x-2">
                <div
                  class="w-8 h-8 rounded-full border-2 border-white shadow-sm ring-2 ring-blue-600"
                  :style="`background-color: ${
                    product.color?.toLowerCase() || 'gray'
                  }`"
                ></div>
              </div>
            </div>

            <!-- Quantity Selector -->
            <div class="py-4 border-b">
              <h3 class="text-sm font-medium text-gray-700">จำนวน</h3>
              <div class="mt-2 flex items-center space-x-2">
                <button
                  @click="decrementQuantity"
                  class="w-8 h-8 rounded-full bg-gray-100 flex items-center justify-center hover:bg-gray-200 transition"
                  :disabled="quantity <= 1"
                >
                  <svg
                    class="w-4 h-4"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M20 12H4"
                    ></path>
                  </svg>
                </button>
                <span class="w-12 text-center font-medium">{{ quantity }}</span>
                <button
                  @click="incrementQuantity"
                  class="w-8 h-8 rounded-full bg-gray-100 flex items-center justify-center hover:bg-gray-200 transition"
                  :disabled="quantity >= product.quantity"
                >
                  <svg
                    class="w-4 h-4"
                    fill="none"
                    stroke="currentColor"
                    viewBox="0 0 24 24"
                  >
                    <path
                      stroke-linecap="round"
                      stroke-linejoin="round"
                      stroke-width="2"
                      d="M12 4v16m8-8H4"
                    ></path>
                  </svg>
                </button>
              </div>
            </div>

            <!-- Action Buttons -->
            <div
              class="py-4 mt-2 flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-2"
            >
              <RouterLink
                :to="{ name: 'Edit', params: { id: product.id } }"
                class="itbms-edit-button w-full sm:w-1/2 bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 transition flex items-center justify-center text-center"
              >
                Edit
              </RouterLink>

              <button
                @click="deleteProduct(product.id)"
                class="itbms-delete-button w-full sm:w-1/2 border border-red-600 text-red-600 py-3 rounded-lg hover:bg-red-50 transition flex items-center justify-center"
              >
                Delete
              </button>
            </div>
          </div>
        </div>
      </div>

      <!------------------------------------------------------------------------------------------------------------------------ -->
      <!-- Product Details Tabs -->
      <div class="mt-8 bg-white rounded-xl shadow-md overflow-hidden">
        <!-- Tabs -->
        <div class="border-b">
          <div class="flex">
            <button
              class="py-4 px-6 font-medium text-sm transition focus:outline-none"
            >
              รายละเอียด
            </button>
          </div>
        </div>

        <!-- Tab Content -->
        <div class="p-6">
          <!-- Specifications Tab -->
          <div class="space-y-6">
            <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
              <!-- Left Column -->
              <div class="space-y-4">
                <div class="bg-gray-50 p-4 rounded-lg">
                  <h3 class="font-medium text-gray-700 mb-2">หน่วยความจำ</h3>
                  <div class="grid grid-cols-3 gap-2">
                    <div class="col-span-1 text-sm text-gray-500">RAM:</div>
                    <div class="col-span-2 text-sm font-medium itbms-ramGb">
                      {{ nullCatching(product.ramGb) }}
                    </div>
                    <span class="itbms-ramGb-unit ml-1 text-lg text-gray-500"
                      >GB</span
                    >

                    <div class="col-span-1 text-sm text-gray-500">Storage:</div>
                    <div class="col-span-2 text-sm font-medium itbms-storageGb">
                      {{ nullCatching(product.storageGb) }} GB
                    </div>
                    <span
                      class="itbms-storageGb-unit ml-1 text-lg text-gray-500"
                      >GB</span
                    >
                  </div>
                </div>

                <div class="bg-gray-50 p-4 rounded-lg">
                  <h3 class="font-medium text-gray-700 mb-2">จอแสดงผล</h3>
                  <div class="grid grid-cols-3 gap-2">
                    <div class="col-span-1 text-sm text-gray-500">ขนาดจอ:</div>
                    <div
                      class="col-span-2 text-sm font-medium itbms-screenSizeInch"
                    >
                      {{ nullCatching(product.screenSizeInch) }}
                    </div>
                    <span
                      class="itbms-screenSizeInch-unit ml-1 text-lg text-gray-500"
                      >Inches</span
                    >
                  </div>
                </div>
              </div>

              <!-- Right Column -->
              <div class="space-y-4">
                <div class="bg-gray-50 p-4 rounded-lg">
                  <h3 class="font-medium text-gray-700 mb-2">ข้อมูลทั่วไป</h3>
                  <div class="grid grid-cols-3 gap-2">
                    <div class="col-span-1 text-sm text-gray-500">แบรนด์:</div>
                    <div class="col-span-2 text-sm font-medium">
                      {{ product.brandName }}
                    </div>

                    <div class="col-span-1 text-sm text-gray-500">รุ่น:</div>
                    <div class="col-span-2 text-sm font-medium">
                      {{ product.model }}
                    </div>

                    <div class="col-span-1 text-sm text-gray-500">สี:</div>
                    <div class="col-span-2 text-sm font-medium">
                      {{ nullCatching(product.color) }}
                    </div>
                  </div>
                </div>
              </div>

              <div class="prose max-w-none">
                <p class="itbms-description text-gray-700">
                  {{ product.description || "ไม่มีข้อมูลรายละเอียดสินค้า" }}
                </p>
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Back Button -->
      <div class="mt-8 flex justify-between items-center">
        <RouterLink to="/sale-items">
          <button
            class="flex items-center text-blue-600 hover:text-blue-800 transition font-medium"
          >
            <svg
              class="w-5 h-5 mr-1"
              fill="none"
              stroke="currentColor"
              viewBox="0 0 24 24"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                stroke-width="2"
                d="M10 19l-7-7m0 0l7-7m-7 7h18"
              ></path>
            </svg>
            กลับไปหน้ารายการสินค้า
          </button>
        </RouterLink>
      </div>
    </div>
  </div>

  <!-- Delete Confirmation Modal -->
  <div
    v-if="showDeleteModal"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black bg-opacity-50"
  >
    <div class="bg-white rounded-lg shadow-lg p-6 w-full max-w-md">
      <h2 class="text-lg font-semibold text-gray-800 mb-4">ยืนยันการลบ</h2>
      <p class="itbms-message text-gray-600 mb-6">
        Do you want to delete this sale item?
      </p>
      <div class="flex justify-end space-x-2">
        <button
          @click="showDeleteModal = false"
          class="itbms-cancel-button px-4 py-2 bg-gray-200 text-gray-800 rounded hover:bg-gray-300 transition"
        >
          ยกเลิก
        </button>
        <button
          @click="confirmDeleteProduct"
          class="itbms-confirm-button px-4 py-2 bg-red-600 text-white rounded hover:bg-red-700 transition"
        >
          ยืนยัน
        </button>
      </div>
    </div>
  </div>

  <!-- Alert Message -->
  <div
    v-if="alertStore.message"
    :class="`itbms-message px-4 py-2 rounded mb-4 ${
      alertStore.type === 'error'
        ? 'bg-red-100 text-red-700'
        : 'bg-green-100 text-green-700'
    }`"
  >
    {{ alertStore.message }}
  </div>
</template>

<style scoped>
/* Additional custom styles can be added here */
</style>
