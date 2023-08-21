<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        Добро пожаловать,
        <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>Сделай заметку</h3>
      <form id="new-todo-form" @submit.prevent="addTodo">
        <h4>Что в вашем списке дел?</h4>
        <input
          type="text"
          name="content"
          id="content"
          placeholder="Что то сделать...? Приготовить завтрак?"
          v-model="inputContent"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              id="category1"
              value="business"
              v-model="inputCategory"
            />
            <span class="bubble business"></span>
            <div>По работе</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              id="category2"
              value="personal"
              v-model="inputCategory"
            />
            <span class="bubble personal"></span>
            <div>Личное</div>
          </label>
        </div>

        <input type="submit" value="Добавить" />
      </form>
    </section>

    <section class="todo-list">
      <h3>Заметки</h3>
      <div class="list">
        <div class="filter-buttons">
          <button
            @click="filterStatus = 'all'"
            :class="{ active: filterStatus === 'all' }"
          >
            Все
          </button>
          <button
            @click="filterStatus = 'completed'"
            :class="{ active: filterStatus === 'completed' }"
          >
            Выполненные
          </button>
          <button
            @click="filterStatus = 'uncompleted'"
            :class="{ active: filterStatus === 'uncompleted' }"
          >
            Невыполненные
          </button>
        </div>
        <div
          v-for="todo in todosArc"
          :key="todo.createdAt"
          :class="`todo-item ${todo.done && 'done'}`"
        >
          <label>
            <input type="checkbox" v-model="todo.done" />
            <span :class="`bubble ${todo.category}`"></span>
          </label>
          <div class="todo-content">
            <input type="text" v-model="todo.content" />
            <p v-if="todo.showCreatedAt" class="created-at">
              {{ formatTime(todo.createdAt) }} -
              {{ formatDate(todo.createdAt) }}
            </p>
          </div>
          <div class="actions">
            <button class="delete" @click="removeTodo(todo)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<script setup>
import { ref, onMounted, computed, watch } from "vue";

const todos = ref([]);
const name = ref(localStorage.getItem("name") || "");

const filterStatus = ref("all");

const inputContent = ref("");
const inputCategory = ref(null);

const todosArc = computed(() => {
  let filteredTodos = todos.value.slice();

  if (filterStatus.value === "completed") {
    filteredTodos = filteredTodos.filter((todo) => todo.done);
  } else if (filterStatus.value === "uncompleted") {
    filteredTodos = filteredTodos.filter((todo) => !todo.done);
  }
  return filteredTodos.sort((a, b) => b.createdAt - a.createdAt);
});

const addTodo = () => {
  if (!inputContent.value.trim() || inputCategory.value === null) {
    return;
  }
  const now = new Date();
  const newTodo = {
    content: inputContent.value,
    category: inputCategory.value,
    done: false,
    createdAt: now.toISOString(),
    showCreatedAt: true,
  };
  todos.value.push(newTodo);
  inputContent.value = "";
  inputCategory.value = null;
  localStorage.setItem("todos", JSON.stringify(todos.value));
};

const formatTime = (isoString) => {
  const time = new Date(isoString);
  const options = {
    hour: "2-digit",
    minute: "2-digit",
  };
  return time.toLocaleTimeString("en-US", options);
};

const formatDate = (isoString) => {
  const date = new Date(isoString);
  const options = {
    year: "numeric",
    month: "short",
    day: "numeric",
  };
  return date.toLocaleDateString("en-US", options);
};

const removeTodo = (todo) => {
  todos.value = todos.value.filter((t) => t !== todo);
};

watch(
  todos,
  (newVal) => {
    localStorage.setItem("todos", JSON.stringify(newVal));
  },
  { deep: true }
);

watch(name, (newVal) => {
  localStorage.setItem("name", newVal);
});

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos")) || [];
});
</script>

<style scoped>
/* ... (other styles) */

.filter-buttons {
  margin: 20px 0;
  display: flex;
  gap: 10px;
}

.filter-buttons button {
  padding: 5px 10px;
  border: 1px solid #ccc;
  background-color: #f0f0f0;
  cursor: pointer;
}

.filter-buttons button.active {
  background-color: #3490dc;
  color: white;
  border-color: #3490dc;
}

/* ... (other styles) */
</style>
