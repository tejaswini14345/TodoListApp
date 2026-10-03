<script setup>
import { computed, ref, watch } from 'vue'

const STORAGE_KEY = 'focuslist.tasks.v1'

const loadTasks = () => {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
  } catch {
    return []
  }
}

const tasks = ref(loadTasks())
const title = ref('')
const priority = ref('medium')
const dueDate = ref('')
const filter = ref('all')
const search = ref('')

watch(
  tasks,
  (value) => localStorage.setItem(STORAGE_KEY, JSON.stringify(value)),
  { deep: true }
)

const remaining = computed(() => tasks.value.filter((task) => !task.completed).length)

const visibleTasks = computed(() => {
  const query = search.value.trim().toLowerCase()

  return tasks.value.filter((task) => {
    const matchesSearch = !query || task.title.toLowerCase().includes(query)
    const matchesFilter =
      filter.value === 'all' ||
      (filter.value === 'active' && !task.completed) ||
      (filter.value === 'completed' && task.completed)

    return matchesSearch && matchesFilter
  })
})

function addTask() {
  const trimmed = title.value.trim()
  if (!trimmed) return

  tasks.value.unshift({
    id: crypto.randomUUID(),
    title: trimmed,
    completed: false,
    priority: priority.value,
    dueDate: dueDate.value || null,
    createdAt: new Date().toISOString()
  })

  title.value = ''
  priority.value = 'medium'
  dueDate.value = ''
}

function toggleTask(id) {
  const task = tasks.value.find((item) => item.id === id)
  if (task) task.completed = !task.completed
}

function removeTask(id) {
  tasks.value = tasks.value.filter((task) => task.id !== id)
}

function clearCompleted() {
  tasks.value = tasks.value.filter((task) => !task.completed)
}

function formatDate(value) {
  if (!value) return ''
  return new Intl.DateTimeFormat(undefined, {
    month: 'short',
    day: 'numeric',
    year: 'numeric'
  }).format(new Date(`${value}T00:00:00`))
}
</script>

<template>
  <main class="shell">
    <section class="hero">
      <div>
        <p class="eyebrow">FocusList</p>
        <h1>Make today feel manageable.</h1>
        <p class="intro">
          A lightweight task manager with priorities, due dates, filters, search, and local persistence.
        </p>
      </div>

      <div class="summary">
        <strong>{{ remaining }}</strong>
        <span>{{ remaining === 1 ? 'task left' : 'tasks left' }}</span>
      </div>
    </section>

    <section class="panel">
      <form class="composer" @submit.prevent="addTask">
        <input
          v-model="title"
          class="task-input"
          aria-label="Task title"
          placeholder="What needs to get done?"
          maxlength="140"
        />

        <div class="composer-row">
          <select v-model="priority" aria-label="Priority">
            <option value="low">Low priority</option>
            <option value="medium">Medium priority</option>
            <option value="high">High priority</option>
          </select>

          <input v-model="dueDate" type="date" aria-label="Due date" />

          <button type="submit">Add task</button>
        </div>
      </form>

      <div class="toolbar">
        <div class="filters" role="group" aria-label="Task filters">
          <button
            v-for="option in ['all', 'active', 'completed']"
            :key="option"
            type="button"
            :class="{ active: filter === option }"
            @click="filter = option"
          >
            {{ option }}
          </button>
        </div>

        <input
          v-model="search"
          class="search"
          type="search"
          placeholder="Search tasks"
          aria-label="Search tasks"
        />
      </div>

      <div v-if="visibleTasks.length" class="task-list">
        <article
          v-for="task in visibleTasks"
          :key="task.id"
          class="task-card"
          :class="{ completed: task.completed }"
        >
          <button
            type="button"
            class="check"
            :aria-label="task.completed ? 'Mark task active' : 'Mark task complete'"
            @click="toggleTask(task.id)"
          >
            <span v-if="task.completed">✓</span>
          </button>

          <div class="task-copy">
            <div class="task-title-row">
              <h2>{{ task.title }}</h2>
              <span class="priority" :data-priority="task.priority">{{ task.priority }}</span>
            </div>
            <p v-if="task.dueDate">Due {{ formatDate(task.dueDate) }}</p>
          </div>

          <button type="button" class="remove" @click="removeTask(task.id)">Remove</button>
        </article>
      </div>

      <div v-else class="empty-state">
        <span>✓</span>
        <h2>No tasks here</h2>
        <p>Add a task or switch filters to see something different.</p>
      </div>

      <footer class="panel-footer">
        <span>{{ tasks.length }} total</span>
        <button type="button" class="link-button" @click="clearCompleted">Clear completed</button>
      </footer>
    </section>
  </main>
</template>
