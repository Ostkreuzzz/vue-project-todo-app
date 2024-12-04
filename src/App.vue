<script setup>
import originalTodos from './data/todos'
import { ref, computed, watch } from 'vue'

const title = ref('')
const errorMessage = ref('')
const todos = ref([])

onBeforeMount(() => {
    try {
        todos.value = JSON.parse(localStorage.getItem('todos'))
    } catch (error) {}

    if (!Array.isArray(todos.value)) {
        todos.value = []
    }
})

const visibleTodos = computed(() => {
    if (status.value === 'active') {
        return activeTodos.value
    }

    if (status.value === 'completed') {
        return todos.value.filter((todo) => todo.completed)
    }

    return todos.value
})

function addTodo() {
    if (!title.value) {
        errorMessage.value = 'Title should not be empty'

        return
    }

    todos.value.push({
        id: Date.now(),
        title: title.value,
        completed: false,
    })

    title.value = ''
}

const activeTodos = computed(() => todos.value.filter((todo) => !todo.completed))

watch(
    todos,
    (newTodos) => {
        localStorage.setItem('todos', JSON.stringify(newTodos))
    },
    { deep: true },
)
</script>

<template>
    <div class="todoapp">
        <h1 class="todoapp__title">todos</h1>

        <div class="todoapp__content">
            <header class="todoapp__header">
                <button
                    type="button"
                    class="todoapp__toggle-all"
                    v-if="!activeTodos.length"
                    :class="{ active: !activeTodos.length }"
                ></button>

                <form @submit.prevent="addTodo">
                    <input
                        data-cy="NewTodoField"
                        type="text"
                        class="todoapp__new-todo"
                        placeholder="What needs to be done?"
                        @input="errorMessage = ''"
                    />
                </form>
            </header>

            <section class="todoapp__main" data-cy="TodoList">
                <div
                    v-for="(todo, i) of visibleTodos"
                    :key="todo.id"
                    class="todo"
                    :class="{ completed: todo.completed }"
                >
                    <label class="todo__status-label">
                        <input type="checkbox" class="todo__status" v-model="todo.completed" />
                    </label>

                    <form v-if="false">
                        <input class="todo__title-field" placeholder="Empty todo will be deleted" v-model="title" />
                    </form>

                    <template v-else>
                        <span class="todo__title">{{ todo.title }}</span>
                        <button class="todo__remove" @click="todos.splice(i, 1)">×</button>
                    </template>

                    <div class="modal overlay" :class="{ 'is-active': false }">
                        <div class="modal-background has-background-white-ter"></div>
                        <div class="loader"></div>
                    </div>
                </div>
            </section>

            <footer class="todoapp__footer">
                <span class="todo-count"> {{ activeTodos.length }} items left </span>

                <nav class="filter">
                    <a href="#/" class="filter__link" :class="{ selected: status === 'all' }" @click="status = 'all'">
                        All
                    </a>
                    <a
                        href="#/active"
                        class="filter__link"
                        :class="{ selected: status === 'active' }"
                        @click="status = 'active'"
                    >
                        Active
                    </a>
                    <a
                        href="#/completed"
                        class="filter__link"
                        :class="{ selected: status === 'completed' }"
                        @click="status = 'completed'"
                    >
                        Completed
                    </a>
                </nav>

                <button
                    type="button"
                    class="todoapp__clear-completed"
                    :disabled="!activeTodos.length"
                    @click="todos = activeTodos"
                >
                    Clear completed
                </button>
            </footer>
        </div>

        <div v-if="errorMessage" class="notification is-danger is-light has-text-weight-normal">
            <button class="delete" @click="errorMessage = ''"></button>

            {{ errorMessage }}
        </div>
    </div>
</template>
