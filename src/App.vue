<template>
  <div class="wrapper">
    <VirtualScroll :total-items="totalItems" @load-more="loadMore">
      <template #default="{ visibleItems, getItemStyle }">
        <div
            v-for="item in visibleItems"
            :key="item"
            class="item"
            :style="getItemStyle(item)"
        >
          <span>Элемент {{ item + 1 }}</span>
          <img src="@/assets/images.jpeg" alt="Item Image" loading="lazy" />
        </div>
      </template>
    </VirtualScroll>
  </div>
</template>

<script setup>
import { ref } from "vue";
import VirtualScroll from "@/components/VirtualScroll.vue";

const initialItems = 10000;
const loadAmount = 10000;

const totalItems = ref(initialItems);

const loadMore = () => {
  totalItems.value += loadAmount;
};
</script>

<style scoped>
.wrapper {
  height: 90vh;
  display: flex;
  flex-direction: column;
}

.item {
  display: flex;
  flex-direction: column;
  box-sizing: border-box;
  padding: 10px;
  background-color: #f0f0f0;
  border: 1px solid #ddd;
}
</style>
