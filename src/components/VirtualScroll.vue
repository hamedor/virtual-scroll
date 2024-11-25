<template>
  <div class="parent">
    <div ref="container" class="container" @scroll="onScroll">
      <div :style="containerStyle">
        <template v-if="visibleItems.length">
          <slot
              :visible-items="visibleItems"
              :get-item-style="getItemStyle"
              :item-width="itemWidth"
              :item-height="itemHeight"
              :gap="dynamicGap"
          ></slot>
        </template>
      </div>
    </div>
  </div>
</template>

<script setup>
import { computed, onMounted, onUnmounted, watch, ref, nextTick } from "vue";

const props = defineProps({
  totalItems: {
    type: Number,
    required: true,
    default: 10000,
  },
  buffer: {
    type: Number,
    default: 5,
  },
  itemWidth: {
    type: Number,
    default: 220,
  },
  itemHeight: {
    type: Number,
    default: 320,
  },
  gap: {
    type: Number,
    default: 13,
  },
  loadThreshold: {
    type: Number,
    default: 100,
  },
});

const emit = defineEmits(['load-more']);

const container = ref(null);

const scrollTop = ref(0);
const scrollLeft = ref(0);

const containerHeight = ref(0);
const containerWidth = ref(0);

const itemsPerRow = computed(() => {
  if (props.itemWidth + props.gap > containerWidth.value) return 1;
  let perRow = Math.floor(containerWidth.value / (props.itemWidth + props.gap));

  while (perRow > 1) {
    const totalItemsWidth = perRow * props.itemWidth;
    const remainingSpace = containerWidth.value - totalItemsWidth;
    const space = remainingSpace / (perRow + 1);

    if (space >= props.gap) break;
    perRow--;
  }

  return perRow;
});

const totalRows = computed(() => {
  return Math.ceil(props.totalItems / itemsPerRow.value);
});

const totalHeight = computed(() => {
  return totalRows.value * (props.itemHeight + props.gap);
});

const itemsInView = computed(() => {
  return (
      Math.ceil(containerHeight.value / (props.itemHeight + props.gap)) *
      itemsPerRow.value
  );
});

const firstVisibleIndex = computed(() => {
  const firstRowIndex = Math.floor(
      scrollTop.value / (props.itemHeight + props.gap)
  );
  return firstRowIndex * itemsPerRow.value;
});

const lastVisibleIndex = computed(() => {
  return (
      firstVisibleIndex.value +
      itemsInView.value +
      props.buffer * itemsPerRow.value
  );
});

const visibleItems = computed(() => {
  const items = [];
  for (
      let i = firstVisibleIndex.value;
      i <= lastVisibleIndex.value && i < props.totalItems;
      i++
  ) {
    items.push(i);
  }
  return items;
});

const containerStyle = computed(() => ({
  height: `${totalHeight.value}px`,
  position: "relative",
  width: "100%",
}));

const dynamicGap = computed(() => {
  if (itemsPerRow.value <= 1) return 0;
  const totalItemsWidth = itemsPerRow.value * props.itemWidth;
  const remainingSpace = containerWidth.value - totalItemsWidth;
  const space = remainingSpace / (itemsPerRow.value + 1);
  return space >= props.gap ? space : props.gap;
});

const getItemStyle = (index) => {
  const rowIndex = Math.floor(index / itemsPerRow.value);
  const colIndex = index % itemsPerRow.value;
  return {
    width: `${props.itemWidth}px`,
    height: `${props.itemHeight}px`,
    position: "absolute",
    top: `${rowIndex * (props.itemHeight + props.gap)}px`,
    left: `${dynamicGap.value * (colIndex + 1) + props.itemWidth * colIndex}px`,
  };
};

const onScroll = () => {
  if (container.value) {
    scrollTop.value = container.value.scrollTop;
    scrollLeft.value = container.value.scrollLeft;
  }
};

const updateContainerSize = () => {
  if (container.value) {
    containerHeight.value = container.value.clientHeight;
    containerWidth.value = container.value.clientWidth;
  }
};

onMounted(async () => {
  await nextTick();
  updateContainerSize();

  window.addEventListener("resize", updateContainerSize);

  onUnmounted(() => {
    window.removeEventListener("resize", updateContainerSize);
  });
});

watch(
    container,
    (newVal) => {
      if (newVal) {
        updateContainerSize();
      }
    },
    { immediate: true }
);

watch(
    visibleItems,
    (newVisibleItems) => {
      const lastVisibleItem = newVisibleItems[newVisibleItems.length - 1];
      if (lastVisibleItem !== undefined && (props.totalItems - lastVisibleItem) <= props.loadThreshold) {
        emit('load-more');
      }
    },
    { immediate: true }
);
</script>

<style scoped>
.parent {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.container {
  flex: 1;
  overflow: auto;
  position: relative;
}
</style>
