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
      <ul class="task-list" style="list-style-type: none;" >
        <li  v-for="(task, index) in sortedTasks"  :key="task.id"  :class="{ completed: task.completed, 'task-item': true }"  >
          <v-row class="align-center">
            <v-col>
              <v-checkbox  v-slot:label class="checkbox" v-model="task.completed" @change="saveTasks" >{{ task.completed ? '☑️' : '⬜' }}</v-checkbox>
            </v-col>
            <v-col cols="7">
              <span v-if="!task.isEditing"  @click="editTask(task)"> {{ task.text }} </span>
              <v-text-field v-else v-model="task.text" class="task-edit-input" @keyup.enter="stopEditing(task)"></v-text-field>
            </v-col>

            <v-col cols="4" class="text-right">
              <v-btn variant="plain" @click="editTask(task) ">✏️</v-btn>
              <v-btn  variant="plain"  @click="deleteTask(index)">🗑️ </v-btn>
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

const newTask = ref('');
const tasks = ref([]);

// Правильно загружаем задачи из localStorage при монтировании
onMounted(() => {
  const savedTasks = JSON.parse(localStorage.getItem('tasks')) || [];
  tasks.value = savedTasks;  // Присваиваем загруженные задачи в реактивное свойство tasks
});

const sortedTasks = computed(() => {
  return tasks.value.slice().sort((a, b) => a.completed - b.completed);
});

const addTask = () => {
  if (newTask.value.trim() === '') {
    alert('Task cannot be empty!');
    return;
  }
  tasks.value.push({
    id: Date.now(),
    text: newTask.value.trim(),
    completed: false,
    isEditing: false
  });
  newTask.value = '';
  saveTasks();  // Убедитесь, что saveTasks вызывается для сохранения изменений
};

const editTask = (task) => {
  task.isEditing = true;  // Исправлена опечатка здесь
};

const deleteTask = (index) => {
  tasks.value.splice(index, 1);
  saveTasks();  // Убедитесь, что saveTasks вызывается для сохранения изменений
};

const stopEditing = (task) => {
  task.isEditing = false;
  saveTasks();  // Убедитесь, что saveTasks вызывается для сохранения изменений
};

const saveTasks = () => {
  localStorage.setItem('tasks', JSON.stringify(tasks.value));
};
</script>
