<template>  
   <div class="kanban-board">
    <div
      class="kanban-column"
      v-for="(column, columnIndex) in columns"
      :key="column.id"
      @dragover.prevent="dragOver(columnIndex)"
      @drop="drop(columnIndex)"
      :class="{ 'drag-over': columnIndex === targetColumn }"
    >
      <h2>{{ column.title }}</h2>
      <div
        class="kanban-card"
        v-for="(card, cardIndex) in column.cards"
        :key="card.id"
        draggable="true"
        @dragstart="dragStart(card, columnIndex, cardIndex)"
        @dragenter.prevent="dragEnter(columnIndex, cardIndex)"
        :class="{ 'dragging': isDragging(card, columnIndex, cardIndex) }"
      >
        {{ card.title }}
      </div>
    </div>
  </div>
  </template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const columns = ref([]);

onMounted(async () => {
  const board = await frappe.call({
    method: 'frappe.client.get_document',
    args: {
      doctype: 'Kanban Board',
      name: 'Teacher Status',
    },
  });

  const columnsData = board.message.columns;

  columnsData.forEach((column) => {
    const cards = await frappe.call({
      method: 'frappe.client.get_list',
      args: {
        doctype: column.reference_doctype,
        filters: {
          [column.field_name]: column.status,
        },
      },
    });

    columns.value.push({
      id: column.name,
      title: column.column_name,
      cards: cards.message.map((card) => ({
        id: card.name,
        title: card[frappe.meta.get_docfield(column.reference_doctype, 'title').label],
      })),
    });
  });
});



let draggedCard = null;
let draggedFromColumn = null;
let draggedFromIndex = null;
let targetColumn = null;
let targetIndex = null;

function dragStart(card, columnIndex, cardIndex) {
  draggedCard = card;
  draggedFromColumn = columnIndex;
  draggedFromIndex = cardIndex;
  targetColumn = null;
  targetIndex = null;
  console.log(frappe.call);
}

function dragEnter(columnIndex, cardIndex) {
  targetColumn = columnIndex;
  targetIndex = cardIndex;
}

function dragOver(columnIndex) {
  if (draggedFromColumn !== null && draggedFromIndex !== null) {
    targetColumn = columnIndex;
    targetIndex = null;
  }
}

function drop(columnIndex) {
  if (draggedCard) {
    if (targetIndex !== null && targetColumn === columnIndex) {
      columns.value[draggedFromColumn].cards.splice(draggedFromIndex, 1);
      columns.value[columnIndex].cards.splice(targetIndex, 0, draggedCard);
    } else {
      columns.value[draggedFromColumn].cards.splice(draggedFromIndex, 1);
      columns.value[columnIndex].cards.push(draggedCard);
    }
  }
  resetDragState();
}

function resetDragState() {
  draggedCard = null;
  draggedFromColumn = null;
  draggedFromIndex = null;
  targetColumn = null;
  targetIndex = null;
}

function isDragging(card, columnIndex, cardIndex) {
  return (
    draggedCard &&
    draggedCard.id === card.id &&
    draggedFromColumn === columnIndex &&
    draggedFromIndex === cardIndex
  );
}
</script>

<style>

</style>
