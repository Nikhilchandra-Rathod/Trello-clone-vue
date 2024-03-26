<script setup lang="ts">
import { nextTick } from "vue";
import { nanoid } from "nanoid";
import draggable from "vuedraggable";
import { useLocalStorage, useKeyModifier, watchDebounced } from "@vueuse/core";
import type { Column, Task } from "../types";

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
</script>

<template>
  <div class="flex items-start gap-4 overflow-x-auto">
    <draggable
      v-model="columns"
      group="columns"
      item-key="id"
      :animation="200"
      handle=".drag-handle"
      class="flex gap-4 items-start pb-10"
    >
      <template #item="{ element: column }: { element: Column }">
        <div class="column bg-gray-200 p-5 rounded min-w-[250px]">
          <header class="font-bold mb-4 flex items-center gap-2">
            <DragHandle />
            <input
              v-model="column.title"
              type="text"
              @keyup.enter="($event.target as HTMLInputElement).blur()"
              @keydown.backspace="column.title === '' ? (columns = columns.filter((c) => c.id !== column.id)) : null"
              class="title-input bg-transparent focus:bg-white rounded px-1 w-4/5"
            />
          </header>

          <draggable
            v-model="column.tasks"
            :group="{ name: 'tasks', pull: alt ? 'clone' : true }"
            item-key="id"
            :animation="200"
            handle=".drag-handle"
          >
            <template #item="{ element: task }: { element: Task }">
              <div>
                <TrelloBoardTask
                  :task="task"
                  @delete="column.tasks = column.tasks.filter((t) => t.id !== $event)"
                />
              </div>
            </template>
          </draggable>

          <footer>
            <NewTask @add="column.tasks.push($event)" />
          </footer>
        </div>
      </template>
    </draggable>

    <button
      @click="createColumn"
      class="bg-gray-200 whitespace-nowrap p-2 rounded opacity-50"
    >
      + Add another column
    </button>
  </div>
</template>
