<script setup lang="ts">
import { ref } from "vue";
import { onKeyStroke } from "@vueuse/core";
import type { Task, ID } from "../types";

import DragHandle from "./DragHandle.vue";

const props = defineProps<{
  task: Task;
}>();

const emit = defineEmits<{
  (e: "delete", payload: ID): void;
}>();

const focused = ref(false);
onKeyStroke("Backspace", (_: KeyboardEvent) => {
  if (focused.value) {
    emit("delete", props.task.id);
  }
});
</script>

<template>
  <div
    :title="task.createdAt.toLocaleDateString()"
    class="task bg-white p-2 mb-2 rounded shadow-sm max-w-[250px] flex items-center"
    @focus="focused = true"
    @blur="focused = false"
    tabindex="0"
  >
    <DragHandle class="pr-2 self-start" />
    <span>{{ task.title }}</span>
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
