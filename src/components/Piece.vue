<script setup lang="ts">
import { ref } from 'vue';

const props = defineProps({
  horizontal: {
    type: Number,
    required: true,
  },
  vertical: {
    type: Number,
    required: true,
  },
  imgUrl: {
    type: String,
    default: "",
  },
});

const isDragging = ref(false);

const size = () => props.horizontal * props.vertical;

const dragStart = (event: DragEvent | any) => {
  isDragging.value = true;
  event.dataTransfer.setData(
    "size",
    JSON.stringify({
      horizontal: props.horizontal,
      vertical: props.vertical,
      size: size(),
      image: props.imgUrl,
    })
  );
};

const dragEnd = () => {
  isDragging.value = false;
};
</script>

<template>
  <div
    class="piece"
    draggable="true"
    @dragstart="dragStart"
    @dragend="dragEnd"
    :class="{ 'is-dragging': isDragging }"
    :style="{ width: `${horizontal}cm`, height: `${vertical}cm` }"
  >
    <div class="drag-handle"></div>
    <img
      :src="imgUrl"
      :style="{ width: `${horizontal}cm`, height: `${vertical}cm` }"
    />
    <div v-for="horiz in horizontal" :key="horiz" class="piece-row">
      <div v-for="vert in vertical" :key="vert" class="piece-cell"></div>
    </div>
  </div>
</template>

<style>
.piece {
  display: flex;
  flex-wrap: wrap;
  position: relative;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
}

.piece img {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
}

.piece-cell {
  width: 1cm;
  height: 1cm;
  /* border: 1px solid black;
  background-color: red; */
}

.drag-handle {
  position: absolute;
  top: -5px;
  left: -5px;
  width: 20px;
  height: 20px;
  background-color: #e2e200;
  cursor: move;
  border-radius: 50px;
  border: 3px solid #989800;
  opacity: 0.8;
}

.drag-handle:hover {
  opacity: 1;
  transform: scale(1.1);
}

.is-dragging {
  opacity: 0.9;
  border: 3px dashed black;
}
</style>