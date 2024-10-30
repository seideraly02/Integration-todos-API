<template>
  <v-container color="">
    <v-card class="pa-4" elevation="2">
      <h1 class="text-center font-weight-bold">ToDo Application</h1>
      <v-row class="align-center">
        <v-col cols="10">
          <v-text-field v-model="newTask" placeholder="Add new task" @keyup.enter="addTask"></v-text-field>
        </v-col>
        <v-col>
          <v-btn variant="outlined" color="blue" @click="addTask" block>+</v-btn>
        </v-col>
      </v-row>
      <ul class="task-list" style="list-style-type: none;">
        <li v-for="(task, index) in sortedTasks" :key="task.id" :class="{ completed: task.completed, 'task-item': true }">
          <v-row class="align-center">
            <v-col>
              <v-checkbox v-slot:label class="checkbox" v-model="task.completed" @change="saveTask(task)">
                {{ task.completed ? '☑️' : '⬜' }}
              </v-checkbox>
            </v-col>
            <v-col cols="7">
              <span v-if="!task.isEditing" @click="editTask(task)"> {{ task.text }} </span>
              <v-text-field v-else v-model="task.text" class="task-edit-input" @keyup.enter="stopEditing(task)"></v-text-field>
            </v-col>
            <v-col cols="4" class="text-right">
              <v-btn variant="plain" @click="deleteTask(task.id, index)">🗑️</v-btn>
            </v-col>
          </v-row>
          <v-divider></v-divider>
        </li>
      </ul>
    </v-card>
  </v-container>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';
import axios from 'axios';

const newTask = ref('');
const tasks = ref([]);

// Загрузка задач из API при монтировании компонента
onMounted(async () => {
  try {
    const response = await axios.get('https://dummyjson.com/todos');
    tasks.value = response.data.todos.map(todo => ({
      id: todo.id,
      text: todo.todo,
      completed: todo.completed,
      isEditing: false,
    }));
  } catch (error) {
    console.error('Ошибка при загрузке задач:', error);
  }
});

const sortedTasks = computed(() => {
  return tasks.value.slice().sort((a, b) => a.completed - b.completed);
});

// Добавление новой задачи
const addTask = async () => {
  if (newTask.value.trim() === '') {
    alert('Task cannot be empty!');
    return;
  }
  try {
    const response = await axios.post('https://dummyjson.com/todos/add', {
      todo: newTask.value.trim(),
      completed: false,
    });
    tasks.value.push({
      id: response.data.id,
      text: response.data.todo,
      completed: response.data.completed,
      isEditing: false,
    });
    newTask.value = '';
  } catch (error) {
    console.error('Ошибка при добавлении задачи:', error);
  }
};

// Включение режима редактирования
const editTask = (task) => {
  task.isEditing = true;
};

// Завершение редактирования и сохранение изменений
const stopEditing = async (task) => {
  task.isEditing = false;
  await saveTask(task);
};

// Сохранение изменений в задаче
const saveTask = async (task) => {
  try {
    await axios.put(`https://dummyjson.com/todos/${task.id}`, {
      todo: task.text,
      completed: task.completed,
    });
  } catch (error) {
    console.error('Ошибка при обновлении задачи:', error);
  }
};

// Удаление задачи
const deleteTask = async (taskId, index) => {
  try {
    await axios.delete(`https://dummyjson.com/todos/${taskId}`);
    tasks.value.splice(index, 1);
  } catch (error) {
    console.error('Ошибка при удалении задачи:', error);
  }
};
</script>

