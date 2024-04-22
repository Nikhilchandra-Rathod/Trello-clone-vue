<script setup lang="ts">
import { nextTick } from "vue";
import { nanoid } from "nanoid";
import draggable from "vuedraggable";
import { useLocalStorage, useKeyModifier, watchDebounced } from "@vueuse/core";
import type { Column, Task } from "../types";
import { IconPlus, IconTrash } from "@tabler/icons-vue";

import NewTask from "./NewTask.vue";
import DragHandle from "./DragHandle.vue";
import TrelloBoardTask from "./TrelloBoardTask.vue";

const columns = useLocalStorage<Column[]>(
  "trelloBoard",
  [
    {
      id: nanoid(),
      title: "Backlog",
      tasks: [
        {
          id: nanoid(),
          title: "Create marketing landing page",
          createdAt: new Date(),
        },
        {
          id: nanoid(),
          title: "Develop cool new feature",
          createdAt: new Date(),
        },
        {
          id: nanoid(),
          title: "Fix page nav bug",
          createdAt: new Date(),
        },
      ],
    },
    {
      id: nanoid(),
      title: "Selected for Dev",
      tasks: [],
    },
    {
      id: nanoid(),
      title: "In Progress",
      tasks: [],
    },
    {
      id: nanoid(),
      title: "QA",
      tasks: [],
    },
    {
      id: nanoid(),
      title: "Complete",
      tasks: [],
    },
  ],
  {
    serializer: {
      read: (value) => {
        return JSON.parse(value).map((column: Column) => {
          column.tasks = column.tasks.map((task: Task) => {
            task.createdAt = new Date(task.createdAt);
            return task;
          });
          return column;
        });
      },
      write: (value) => JSON.stringify(value),
    },
  }
);
const alt = useKeyModifier("Alt");

function createColumn() {
  columns.value.push({
    id: nanoid(),
    title: "",
    tasks: [],
  });
  nextTick(() => {
    (document.querySelector(".column:last-of-type .title-input") as HTMLInputElement).focus();
  });
}

watchDebounced(
  columns,
  () => {
    console.log("something changed");
  },
  {
    deep: true,
    debounce: 500,
    maxWait: 1000,
  }
);

function handleDeleteColumn(column: Column) {
  columns.value = columns.value.filter((c) => c.id !== column.id);
}
</script>

<template>
  <div class="flex items-center px-4 py-4 justify-end">
    <button
      @click="createColumn"
      class="bg-blue-900 text-blue-50 whitespace-nowrap py-2 px-4 rounded flex items-center gap-1"
    >
      <IconPlus :size="20" /> Add another column
    </button>
  </div>
  <draggable
    v-model="columns"
    group="columns"
    item-key="id"
    :animation="200"
    handle=".drag-handle"
    class="px-4 flex flex-1 items-start gap-4 overflow-x-auto"
  >
    <template #item="{ element: column }: { element: Column }">
      <div class="column bg-gray-200 p-5 rounded min-w-[350px] flex-1 h-full overflow-hidden pb-40">
        <header class="font-bold mb-4 flex items-center gap-2 justify-between">
          <div class="flex items-center">
            <DragHandle />
            <input
              v-model="column.title"
              type="text"
              @keyup.enter="($event.target as HTMLInputElement).blur()"
              @keydown.backspace="column.title === '' ? (columns = columns.filter((c) => c.id !== column.id)) : null"
              class="title-input bg-transparent focus:bg-white rounded px-1 w-4/5"
            />
          </div>
          <div>
            <button
              @click="handleDeleteColumn(column)"
              class="flex items-center justify-center cursor-pointer text-red-500 hover:bg-red-500 hover:text-white rounded-sm w-5 h-5"
            >
              <IconTrash :size="15" />
            </button>
          </div>
        </header>

        <draggable
          v-model="column.tasks"
          :group="{ name: 'tasks', pull: alt ? 'clone' : true }"
          item-key="id"
          :animation="200"
          handle=".drag-handle"
          class="flex-1 overflow-y-auto h-full"
        >
          <template #item="{ element: task }: { element: Task }">
            <TrelloBoardTask
              :task="task"
              @delete="column.tasks = column.tasks.filter((t) => t.id !== $event)"
            />
          </template>
        </draggable>

        <footer class="mt-4">
          <NewTask @add="column.tasks.push($event)" />
        </footer>
      </div>
    </template>
  </draggable>
</template>
