<script setup lang="ts">
import { ref } from "vue";
import { nanoid } from "nanoid";
import type { Task } from "../types";

const emit = defineEmits<{
  (e: "add", payload: Task): void;
}>();

const focused = ref(false);
const title = ref("");

function createTask(e: Event) {
  if (title.value.trim()) {
    e.preventDefault();
    emit("add", {
      title: title.value.trim(),
      createdAt: new Date(),
      id: nanoid(),
    });
  }

  title.value = "";
}
</script>

<template>
  <div>
    <textarea
      v-model="title"
      @keydown.tab="createTask"
      @keydown.enter="createTask"
      class="focus:bg-white focus:shadow resize-none rounded w-full border bg-transparent focus:p-2"
      :class="{
        'h-7': !focused,
        'h-20': focused,
      }"
      style="outline: none !important"
      @focus="focused = true"
      @blur="focused = false"
      :placeholder="!focused ? '+ Add a Card' : 'Enter a title for this card'"
    ></textarea>
  </div>
</template>
