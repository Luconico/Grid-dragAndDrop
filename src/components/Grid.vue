<script setup lang="ts">
import { ref, Ref } from 'vue';
import Piece from './Piece.vue';
import html2canvas from 'html2canvas';

interface PieceData {
  horizontal: number;
  vertical: number;
  image: string;
}

type PieceDataWithId = PieceData & { id: string };

interface Cell {
  filled: boolean;
  piece: PieceDataWithId | null;
}

const gridRef = ref(null);
const showHandle = ref(true);
const toRemove = ref(false);

const grid: Ref<Cell[][]> = ref(
  Array(20)
    .fill(null)
    .map(() =>
      Array(20)
        .fill(null)
        .map(() => ({ filled: false, piece: null }))
    )
);

const actionHistory = ref<Array<{ rowIndex: number, cellIndex: number, piece: PieceDataWithId | null }>>([]);

const drop = (event: DragEvent, rowIndex: number, cellIndex: number) => {
  toRemove.value = false;

  const sizeData = JSON.parse(event.dataTransfer?.getData('size') || '{}');

  const { horizontal, vertical } = sizeData;

  if (horizontal > grid.value[0].length || vertical > grid.value.length) {
    return;
  }

  for (let i = 0; i < horizontal; i++) {
    for (let j = 0; j < vertical; j++) {
      if (grid.value[rowIndex + j][cellIndex + i].filled) {
        return;
      }
    }
  }

  toRemove.value = true;

  for (let i = 0; i < horizontal; i++) {
    for (let j = 0; j < vertical; j++) {
      grid.value[rowIndex + j][cellIndex + i].filled = true;
    }
  }

  const pieceWithId: PieceDataWithId = {
    ...sizeData,
    id: `${rowIndex}-${cellIndex}`,
  }

  grid.value[rowIndex][cellIndex].piece = pieceWithId;

  actionHistory.value.push({
    rowIndex,
    cellIndex,
    piece: pieceWithId,
  });
};

const undo = () => {
  const lastAction = actionHistory.value.pop();
  if (lastAction) {
    const { rowIndex, cellIndex, piece } = lastAction;
    if (piece) {
      for (let i = 0; i < piece.horizontal; i++) {
        for (let j = 0; j < piece.vertical; j++) {
          if (
            cellIndex + i < grid.value[0].length &&
            rowIndex + j < grid.value.length
          ) {
            grid.value[rowIndex + j][cellIndex + i].filled = false;
            grid.value[rowIndex + j][cellIndex + i].piece = null;
          }
        }
      }
    }
  }
};

const removePieze = (rowIndex: any, cellIndex: any, piece: any) => {
  if (!piece || !toRemove.value) return;
  for (let i = 0; i < piece.horizontal; i++) {
    for (let j = 0; j < piece.vertical; j++) {
      if (
        cellIndex + i < grid.value[0].length &&
        rowIndex + j < grid.value.length
      ) {
        grid.value[rowIndex + j][cellIndex + i].filled = false;
        grid.value[rowIndex + j][cellIndex + i].piece = null;
      }
    }
  }
}

const takeScreenshot = () => {
  showHandle.value = false;
  setTimeout(() => {
    if (!gridRef.value) return;
    html2canvas(gridRef.value).then(canvas => {
      let pngUrl = canvas.toDataURL("image/png");
      let downloadLink = document.createElement("a");
      downloadLink.href = pngUrl;
      downloadLink.download = 'imagen.png';
      document.body.appendChild(downloadLink);
      downloadLink.click();
      document.body.removeChild(downloadLink);
    }).finally(() => {
      showHandle.value = true;
    })
  }, 1);
};

</script>

<template>
  <button @click="undo">Undo</button><button @click="takeScreenshot">IMG busy-board</button>
  <div class="grid-container">
    <div class="grid" ref="gridRef">
      <div v-for="(row, rowIndex) in grid" :key="rowIndex" class="row">
        <div v-for="(cell, cellIndex) in row" :key="cellIndex" class="cell" :class="{ 'cell-filled': cell.filled }"
          @dragover.prevent @drop="drop($event, rowIndex, cellIndex)" :id="`cell-${rowIndex}-${cellIndex}`">
          <Piece v-if="cell.piece" :horizontal="cell.piece.horizontal" :vertical="cell.piece.vertical"
            :imgUrl="cell.piece.image" :showHandle="showHandle" @dragend="removePieze(rowIndex, cellIndex, cell.piece)" />
        </div>
      </div>
    </div>
    <div>
      <Piece :horizontal="6" :vertical="6" :imgUrl="'moto.png'" />
      <hr />
      <Piece :horizontal="4" :vertical="8" :imgUrl="'cohete.png'" />
    </div>
  </div>
</template>

<style>
.cell {
  width: 1cm;
  height: 1cm;
  border: 1px solid black;
  position: relative;
}

.cell .piece {
  position: absolute;
  top: 0;
  left: 0;
}

.row {
  display: flex;
}

.grid-container {
  display: flex;
  gap: 1rem;
}
</style>