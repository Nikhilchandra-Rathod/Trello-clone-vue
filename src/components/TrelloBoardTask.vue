<script setup lang="ts">
import { nextTick, ref } from "vue";
import { onKeyStroke } from "@vueuse/core";
import type { Task, ID } from "../types";

import DragHandle from "./DragHandle.vue";
import { IconTrash } from "@tabler/icons-vue";

const props = defineProps<{
  task: Task;
}>();

const emit = defineEmits<{
  (e: "delete", payload: ID): void;
}>();

const handleDelete = () => {
  emit("delete", props.task.id);
};

const inputRef = ref<HTMLInputElement | null>(null);
const isEdit = ref(false);

const handleEdit = () => {
  isEdit.value = true;
  nextTick(() => {
    inputRef.value?.focus();
  });
};
</script>

<template>
  <div
    :title="task.createdAt.toLocaleDateString()"
    class="task bg-white p-2 mb-2 rounded shadow-sm max-w-[350px] flex items-center justify-between"
  >
    <div class="flex items-center w-full">
      <DragHandle class="pr-2 self-start" />
      <span
        v-if="!isEdit"
        @click="handleEdit"
        class="flex items-center flex-1"
        >{{ task.title }}</span
      >
      <input
        ref="inputRef"
        v-if="isEdit"
        v-model="task.title"
        type="text"
        @keyup.enter="($event.target as HTMLInputElement).blur()"
        @blur="isEdit = false"
        class="title-input flex-1 bg-transparent focus:bg-white rounded w-4/5"
      />
    </div>
    <div>
      <button
        @click="handleDelete"
        class="flex items-center justify-center cursor-pointer text-red-500 hover:bg-red-500 hover:text-white rounded-sm w-5 h-5"
      >
        <IconTrash :size="15" />
      </button>
    </div>
  </div>
</template>

<style>
.sortable-drag .task {
  transform: rotate(5deg);
}

.sorgable-ghost .task {
  position: relative;
}

.sorgable-ghost .task::after {
  content: "";
  @apply absolute top-0 bottom-0 left-0 right-0 bg-slate-300 rounded;
}

.task:focus,
.task:focus-visible {
  @apply outline-gray-400 !important;
  outline: gray auto 1px;
}
</style>
