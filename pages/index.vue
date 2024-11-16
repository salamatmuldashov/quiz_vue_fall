<template>
  <div class="todo-list">
    <div class="task-input">
      <input v-model="newTaskTitle" type="text" placeholder="New Task" />
      <select v-model="newTaskPriority">
        <option value="low">Low</option>
        <option value="medium">Medium</option>
        <option value="high">High</option>
      </select>
      <button @click="addTask">Add Task</button>
    </div>

    <p>Pending tasks: {{ pendingTasksCount }}</p>

    <transition-group name="task" tag="ul">
      <li
        v-for="task in sortedTasks"
        :key="task.id"
        :class="['task', task.priority, { completed: task.completed }]"
      >
        <span class="task-title">{{ task.title }}</span>
        <div class="buttons">
          <button @click.stop="toggleCompletion(task)" class="done">
            Done
          </button>
          <button @click.stop="removeTask(task.id)">Remove</button>
        </div>
      </li>
    </transition-group>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, nextTick, watch } from "vue";

interface Task {
  id: number;
  title: string;
  priority: string;
  completed: boolean;
}

const tasks = ref<Task[]>([]);
const newTaskTitle = ref<string>("");
const newTaskPriority = ref<string>("low");
const pendingTasksCount = computed(
  () => tasks.value.filter((task) => !task.completed).length
);
const sortedTasks = computed(() => {
  return [...tasks.value].sort((a, b) => {
    const priorities = ["low", "medium", "high"];
    return priorities.indexOf(a.priority) - priorities.indexOf(b.priority);
  });
});

const addTask = () => {
  if (newTaskTitle.value.trim()) {
    const newTask: Task = {
      id: Date.now(),
      title: newTaskTitle.value,
      priority: newTaskPriority.value,
      completed: false,
    };
    tasks.value.push(newTask);
    newTaskTitle.value = "";
    newTaskPriority.value = "low";
    nextTick(() => {
      const lastTask = document.querySelector("li:last-child");
      if (lastTask) {
        lastTask.scrollIntoView({ behavior: "smooth" });
      }
    });
  }
};

const removeTask = (taskId: number) => {
  tasks.value = tasks.value.filter((task) => task.id !== taskId);
};

const toggleCompletion = (task: Task) => {
  task.completed = !task.completed;
};
watch(tasks, (newTasks, oldTasks) => {
  if (newTasks.length > oldTasks.length) {
    window.alert("A new task was added.");
  }
  if (newTasks.some((task) => task.completed)) {
    window.alert("A task has been completed.");
  }
});
</script>

<style scoped>
.task {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px;
  margin: 5px 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}
.buttons{
    display: flex;
    gap: 2rem;
}
.task.low {
  background-color: lightgreen;
}

.task.medium {
  background-color: lightyellow;
}

.task.high {
  background-color: lightcoral;
}

.task.completed .task-title {
  text-decoration: line-through;
  color: gray;
}

.task-input input,
.task-input select {
  margin-right: 10px;
}

.task-input button {
  background-color: #4caf50;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}
.done {
  background-color: #4caf50;
}
.task-input button:hover {
  background-color: #45a049;
}

button {
  background-color: #f44336;
  color: white;
  border: none;
  padding: 5px 10px;
  cursor: pointer;
}

button:hover {
  background-color: #e53935;
}

.task-enter-active,
.task-leave-active {
  transition: opacity 0.5s ease;
}
.task-enter,
.task-leave-to {
  opacity: 0;
}
</style>
