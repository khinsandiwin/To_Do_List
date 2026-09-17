<script setup>
import { computed, onMounted, ref, watch } from 'vue'

// localStorage keys are kept in one place so the persistence behavior is easy to change later.
const TASKS_KEY = 'minimal-todos'
const THEME_KEY = 'minimal-todos-theme'

// Category chips give each task a simple visual tag without requiring a separate settings screen.
const categories = [
  { name: 'Work', classes: 'bg-sky-100 text-sky-700 dark:bg-sky-400/15 dark:text-sky-200' },
  { name: 'Personal', classes: 'bg-emerald-100 text-emerald-700 dark:bg-emerald-400/15 dark:text-emerald-200' },
  { name: 'Urgent', classes: 'bg-rose-100 text-rose-700 dark:bg-rose-400/15 dark:text-rose-200' },
]

const filters = ['All', 'Active', 'Completed']

const newTaskTitle = ref('')
const selectedCategory = ref(categories[0].name)
const activeFilter = ref('All')
const tasks = ref([])
const editingId = ref(null)
const editingTitle = ref('')
const isDark = ref(false)

// Derived task counts keep the template readable and power both the tabs and progress bar.
const completedCount = computed(() => tasks.value.filter((task) => task.completed).length)
const activeCount = computed(() => tasks.value.length - completedCount.value)
const progressPercent = computed(() =>
  tasks.value.length ? Math.round((completedCount.value / tasks.value.length) * 100) : 0,
)

const filteredTasks = computed(() => {
  if (activeFilter.value === 'Active') {
    return tasks.value.filter((task) => !task.completed)
  }

  if (activeFilter.value === 'Completed') {
    return tasks.value.filter((task) => task.completed)
  }

  return tasks.value
})

const emptyStateMessage = computed(() => {
  if (activeFilter.value === 'Active') return 'No active tasks. Enjoy the quiet moment.'
  if (activeFilter.value === 'Completed') return 'No completed tasks yet. One small win starts it.'
  return 'Your list is clear. Add a task when you are ready.'
})

const addTask = () => {
  const title = newTaskTitle.value.trim()

  if (!title) return

  tasks.value.unshift({
    id: crypto.randomUUID(),
    title,
    category: selectedCategory.value,
    completed: false,
    createdAt: new Date().toISOString(),
  })

  newTaskTitle.value = ''
}

const toggleTask = (taskId) => {
  const task = tasks.value.find((item) => item.id === taskId)
  if (task) task.completed = !task.completed
}

const deleteTask = (taskId) => {
  tasks.value = tasks.value.filter((task) => task.id !== taskId)
}

const startEditing = (task) => {
  editingId.value = task.id
  editingTitle.value = task.title
}

const saveEdit = (taskId) => {
  const title = editingTitle.value.trim()

  if (!title) {
    deleteTask(taskId)
  } else {
    const task = tasks.value.find((item) => item.id === taskId)
    if (task) task.title = title
  }

  editingId.value = null
  editingTitle.value = ''
}

const cancelEdit = () => {
  editingId.value = null
  editingTitle.value = ''
}

const categoryClasses = (categoryName) =>
  categories.find((category) => category.name === categoryName)?.classes ?? categories[0].classes

const toggleTheme = () => {
  isDark.value = !isDark.value
}

// Load saved tasks and the user's theme preference once the browser APIs are available.
onMounted(() => {
  const savedTasks = localStorage.getItem(TASKS_KEY)
  const savedTheme = localStorage.getItem(THEME_KEY)

  tasks.value = savedTasks
    ? JSON.parse(savedTasks)
    : [
        {
          id: crypto.randomUUID(),
          title: 'Review project priorities',
          category: 'Work',
          completed: false,
          createdAt: new Date().toISOString(),
        },
        {
          id: crypto.randomUUID(),
          title: 'Plan a screen-free evening',
          category: 'Personal',
          completed: true,
          createdAt: new Date().toISOString(),
        },
      ]

  isDark.value = savedTheme
    ? savedTheme === 'dark'
    : window.matchMedia('(prefers-color-scheme: dark)').matches
})

// Persist task changes deeply, including edits and completion toggles.
watch(
  tasks,
  (nextTasks) => {
    localStorage.setItem(TASKS_KEY, JSON.stringify(nextTasks))
  },
  { deep: true },
)

// Tailwind's class-based dark mode listens for the `dark` class on the root element.
watch(
  isDark,
  (enabled) => {
    document.documentElement.classList.toggle('dark', enabled)
    localStorage.setItem(THEME_KEY, enabled ? 'dark' : 'light')
  },
  { immediate: true },
)
</script>

<template>
  <main
    class="min-h-screen bg-zinc-100 px-4 py-8 text-zinc-950 transition-colors duration-300 dark:bg-zinc-950 dark:text-zinc-50 sm:px-6 lg:px-8"
  >
    <section class="mx-auto flex min-h-[calc(100vh-4rem)] w-full max-w-3xl items-center">
      <div
        class="w-full rounded-2xl border border-white/70 bg-white/85 p-5 shadow-2xl shadow-zinc-200/80 backdrop-blur transition-colors duration-300 dark:border-zinc-800 dark:bg-zinc-900/85 dark:shadow-black/30 sm:p-7"
      >
        <header class="mb-7 flex flex-col gap-5 sm:flex-row sm:items-start sm:justify-between">
          <div>
            <p class="text-sm font-medium uppercase tracking-[0.2em] text-zinc-500 dark:text-zinc-400">
              Khin's Daily Tasks
            </p>
            <h1 class="mt-2 text-3xl font-semibold tracking-tight sm:text-4xl">Today&apos;s focus</h1>
            <p class="mt-2 text-sm text-zinc-500 dark:text-zinc-400">
              {{ completedCount }} of {{ tasks.length }} completed
            </p>
          </div>

          <button
            type="button"
            class="group inline-flex h-11 w-20 items-center rounded-full border border-zinc-200 bg-zinc-100 p-1 transition duration-300 hover:scale-[1.02] hover:border-zinc-300 dark:border-zinc-700 dark:bg-zinc-800"
            :aria-label="isDark ? 'Switch to light mode' : 'Switch to dark mode'"
            @click="toggleTheme"
          >
            <span
              class="grid h-9 w-9 place-items-center rounded-full bg-white text-sm shadow-sm transition duration-300 group-hover:shadow-md dark:translate-x-9 dark:bg-zinc-950"
            >
              {{ isDark ? '☾' : '☀' }}
            </span>
          </button>
        </header>

        <div class="mb-7">
          <div class="mb-2 flex items-center justify-between text-sm text-zinc-500 dark:text-zinc-400">
            <span>Progress</span>
            <span>{{ progressPercent }}%</span>
          </div>
          <div class="h-3 overflow-hidden rounded-full bg-zinc-200 dark:bg-zinc-800">
            <div
              class="h-full rounded-full bg-gradient-to-r from-emerald-400 to-sky-500 transition-all duration-500"
              :style="{ width: `${progressPercent}%` }"
            ></div>
          </div>
        </div>

        <form class="mb-6 grid gap-3 sm:grid-cols-[1fr_auto_auto]" @submit.prevent="addTask">
          <input
            v-model="newTaskTitle"
            type="text"
            placeholder="Add a new task..."
            class="h-12 rounded-2xl border border-zinc-200 bg-white px-4 text-sm outline-none transition focus:border-sky-400 focus:ring-4 focus:ring-sky-100 dark:border-zinc-700 dark:bg-zinc-950 dark:focus:border-sky-500 dark:focus:ring-sky-500/10"
          />

          <select
            v-model="selectedCategory"
            class="h-12 rounded-2xl border border-zinc-200 bg-white px-4 text-sm outline-none transition focus:border-sky-400 focus:ring-4 focus:ring-sky-100 dark:border-zinc-700 dark:bg-zinc-950 dark:focus:border-sky-500 dark:focus:ring-sky-500/10"
          >
            <option v-for="category in categories" :key="category.name" :value="category.name">
              {{ category.name }}
            </option>
          </select>

          <button
            type="submit"
            class="h-12 rounded-2xl bg-zinc-950 px-5 text-sm font-semibold text-white shadow-lg shadow-zinc-300 transition hover:-translate-y-0.5 hover:bg-zinc-800 active:translate-y-0 dark:bg-white dark:text-zinc-950 dark:shadow-black/30 dark:hover:bg-zinc-200"
          >
            Add
          </button>
        </form>

        <nav class="mb-5 grid grid-cols-3 gap-2 rounded-2xl bg-zinc-100 p-1 dark:bg-zinc-950">
          <button
            v-for="filter in filters"
            :key="filter"
            type="button"
            class="rounded-xl px-3 py-2 text-sm font-medium transition"
            :class="
              activeFilter === filter
                ? 'bg-white text-zinc-950 shadow-sm dark:bg-zinc-800 dark:text-white'
                : 'text-zinc-500 hover:text-zinc-950 dark:text-zinc-400 dark:hover:text-white'
            "
            @click="activeFilter = filter"
          >
            {{ filter }}
            <span v-if="filter === 'Active'">({{ activeCount }})</span>
            <span v-else-if="filter === 'Completed'">({{ completedCount }})</span>
          </button>
        </nav>

        <TransitionGroup name="task" tag="ul" class="space-y-3">
          <li
            v-for="task in filteredTasks"
            :key="task.id"
            class="rounded-2xl border border-zinc-200 bg-white p-4 shadow-sm transition duration-200 hover:-translate-y-0.5 hover:shadow-md dark:border-zinc-800 dark:bg-zinc-950"
          >
            <div class="flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
              <div class="flex min-w-0 flex-1 items-start gap-3">
                <button
                  type="button"
                  class="mt-0.5 grid h-6 w-6 shrink-0 place-items-center rounded-full border transition"
                  :class="
                    task.completed
                      ? 'border-emerald-500 bg-emerald-500 text-white'
                      : 'border-zinc-300 hover:border-emerald-500 dark:border-zinc-600'
                  "
                  :aria-label="task.completed ? 'Mark task active' : 'Mark task complete'"
                  @click="toggleTask(task.id)"
                >
                  <span v-if="task.completed" class="text-xs">✓</span>
                </button>

                <div class="min-w-0 flex-1">
                  <input
                    v-if="editingId === task.id"
                    v-model="editingTitle"
                    type="text"
                    class="w-full rounded-xl border border-sky-300 bg-white px-3 py-2 text-sm outline-none ring-4 ring-sky-100 dark:border-sky-500 dark:bg-zinc-900 dark:ring-sky-500/10"
                    @keyup.enter="saveEdit(task.id)"
                    @keyup.esc="cancelEdit"
                    @blur="saveEdit(task.id)"
                  />
                  <p
                    v-else
                    class="break-words text-base font-medium transition"
                    :class="task.completed ? 'text-zinc-400 line-through opacity-70' : 'text-zinc-900 dark:text-zinc-100'"
                  >
                    {{ task.title }}
                  </p>

                  <span
                    class="mt-2 inline-flex rounded-full px-2.5 py-1 text-xs font-semibold"
                    :class="categoryClasses(task.category)"
                  >
                    {{ task.category }}
                  </span>
                </div>
              </div>

              <div class="flex items-center gap-2 sm:justify-end">
                <button
                  type="button"
                  class="rounded-xl px-3 py-2 text-sm font-medium text-zinc-500 transition hover:bg-zinc-100 hover:text-zinc-950 dark:text-zinc-400 dark:hover:bg-zinc-900 dark:hover:text-white"
                  @click="startEditing(task)"
                >
                  Edit
                </button>
                <button
                  type="button"
                  class="rounded-xl px-3 py-2 text-sm font-medium text-rose-500 transition hover:bg-rose-50 dark:hover:bg-rose-500/10"
                  @click="deleteTask(task.id)"
                >
                  Delete
                </button>
              </div>
            </div>
          </li>
        </TransitionGroup>

        <div
          v-if="filteredTasks.length === 0"
          class="mt-6 rounded-2xl border border-dashed border-zinc-300 p-8 text-center dark:border-zinc-700"
        >
          <div class="mx-auto mb-4 grid h-16 w-16 place-items-center rounded-2xl bg-zinc-100 text-3xl dark:bg-zinc-800">
            ✦
          </div>
          <h2 class="text-lg font-semibold">Nothing here</h2>
          <p class="mt-2 text-sm text-zinc-500 dark:text-zinc-400">{{ emptyStateMessage }}</p>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
/* TransitionGroup hooks add a soft, modern feel when tasks enter or leave the list. */
.task-enter-active,
.task-leave-active {
  transition:
    opacity 180ms ease,
    transform 180ms ease;
}

.task-enter-from,
.task-leave-to {
  opacity: 0;
  transform: translateY(8px);
}
</style>
