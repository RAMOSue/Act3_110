<template>
  <div class="todo-container">
    <h2 class="title">TO DO LIST</h2>

    <!-- Form to Add a Task -->
    <form @submit.prevent="addTask" class="task-form">
      <input
        v-model="newTask"
        type="text"
        placeholder="Add a new task"
        required
        class="task-input"
      />
      <button type="submit" class="add-task-button">Add Task</button>
    </form>

    <!-- Display Tasks -->
    <ul class="task-list">
      <li v-for="task in tasks" :key="task.id" class="task-item">
        <input
          type="checkbox"
          v-model="task.completed"
          @change="updateTask(task)"
          class="task-checkbox"
        />
        <span :class="{ completed: task.completed }" class="task-text">
          {{ task.task }}
        </span>
        <div class="button-group">
          <button @click="deleteTask(task.id)" class="delete-button">Delete</button>
          <button @click="openEditModal(task)" class="edit-button">Edit</button>
        </div>
      </li>
    </ul>

    <!-- Confirmation Modal for Editing Task -->
    <div v-if="isModalVisible" class="modal-overlay">
      <div class="modal">
        <h3>Update Task</h3>
        <input v-model="editedTask" type="text" class="modal-input" />
        <button @click="confirmEditTask" class="confirm-button">Confirm</button>
        <button @click="closeModal" class="cancel-button">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      tasks: [],
      newTask: "",
      editedTask: "",
      taskToEdit: null,
      isModalVisible: false,
    };
  },
  mounted() {
    this.loadTasksFromLocalStorage();
    this.fetchTasks();
  },
  methods: {
    validateInput(input) {
      const forbiddenPatterns = /<[^>]+>|[{}[\]()<>;*%$#@!&]/g;
      if (forbiddenPatterns.test(input)) {
        alert("Your input contains invalid characters or scripts.");
        return false;
      }
      return true;
    },
    loadTasksFromLocalStorage() {
      const storedTasks = localStorage.getItem("tasks");
      if (storedTasks) {
        this.tasks = JSON.parse(storedTasks);
      }
    },
    saveTasksToLocalStorage() {
      localStorage.setItem("tasks", JSON.stringify(this.tasks));
    },
    fetchTasks() {
      axios.get("/api/tasks")
        .then(response => {
          this.tasks = response.data;
          this.saveTasksToLocalStorage();
        })
        .catch(error => console.error("Error fetching tasks:", error));
    },
    addTask() {
      if (!this.newTask.trim()) return;
      if (!this.validateInput(this.newTask)) return;

      const newTask = {
        id: Date.now(),
        task: this.newTask,
        completed: false,
      };

      this.tasks.push(newTask);
      this.saveTasksToLocalStorage();
      this.newTask = "";

      axios.post("/api/tasks", { task: newTask.task })
        .then(response => {
          newTask.id = response.data.id;
          this.saveTasksToLocalStorage();
        })
        .catch(error => console.error("Error adding task to backend:", error));
    },
    updateTask(task) {
      this.saveTasksToLocalStorage();
      axios.patch(`/api/tasks/${task.id}`, { completed: task.completed })
        .catch(error => console.error("Error updating task:", error));
    },
    deleteTask(id) {
      this.tasks = this.tasks.filter(task => task.id !== id);
      this.saveTasksToLocalStorage();
      axios.delete(`/api/tasks/${id}`)
        .catch(error => console.error("Error deleting task:", error));
    },
    openEditModal(task) {
      this.editedTask = task.task;
      this.taskToEdit = task;
      this.isModalVisible = true;
    },
    closeModal() {
      this.isModalVisible = false;
      this.editedTask = "";
      this.taskToEdit = null;
    },
    confirmEditTask() {
      if (this.editedTask.trim() && this.taskToEdit) {
        this.taskToEdit.task = this.editedTask.trim();
        this.saveTasksToLocalStorage();

        axios.patch(`/api/tasks/${this.taskToEdit.id}`, { task: this.taskToEdit.task })
          .then(() => this.saveTasksToLocalStorage())
          .catch(error => console.error("Error updating task in backend:", error));

        this.closeModal();
      }
    },
  },
};
</script>

<style scoped>
/* Full-Screen Layout */
body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  font-family: 'Arial', sans-serif;
}

.todo-container {
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: flex-start;
  align-items: center;
  padding: 20px;
  background: linear-gradient(135deg, #b4e3f1, #b2c7d9);
  box-sizing: border-box;
}

.title {
  font-size: 3rem;
  color: #1e3c5d;
  margin-bottom: 20px;
  text-align: center;
}

.task-form {
  display: flex;
  gap: 12px;
  margin-bottom: 30px;
  width: 100%;
  max-width: 700px;
  margin: 0 auto;
}

.task-input {
  flex-grow: 1;
  padding: 15px;
  font-size: 1.2rem;
  border: 2px solid #88c6d3;
  border-radius: 8px;
  width: 100%;
  max-width: 500px;
  box-sizing: border-box;
}

.task-input:focus {
  border-color: #4fa3c1;
}

.add-task-button {
  padding: 14px 20px;
  background: linear-gradient(135deg, #57d2c7, #40b7b1);
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  cursor: pointer;
  transition: transform 0.2s, background-color 0.3s;
}

.add-task-button:hover {
  background: linear-gradient(135deg, #52a2b1, #409ca4);
  transform: scale(1.05);
}

.task-list {
  list-style: none;
  padding: 0;
  margin: 0;
  width: 100%;
  max-width: 700px;
  margin: 0 auto;
}

.task-item {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px;
  border-bottom: 2px solid #e0e8f2;
  font-size: 1.1rem;
  transition: background-color 0.3s;
  border-radius: 8px;
}

.task-item:hover {
  background-color: #d6e7f0;
}

.task-checkbox {
  margin-right: 20px;
}

.task-text {
  flex-grow: 1;
  color: #324b61;
  transition: text-decoration 0.3s ease;
}

.completed {
  text-decoration: line-through;
  color: #a3a3a3;
}

.button-group {
  display: flex;
  gap: 12px;
}

.delete-button,
.edit-button {
  padding: 10px 15px;
  font-size: 1rem;
  border-radius: 6px;
  cursor: pointer;
  transition: transform 0.2s, background-color 0.3s;
}

.delete-button {
  background: linear-gradient(135deg, #e57f7f, #d64d4d);
  color: white;
}

.delete-button:hover {
  background: linear-gradient(135deg, #ff6a6a, #ff4b4b);
  transform: scale(1.05);
}

.edit-button {
  background: linear-gradient(135deg, #88b6ff, #557bdb);
  color: white;
}

.edit-button:hover {
  background: linear-gradient(135deg, #6e99e6, #4f6fbb);
  transform: scale(1.05);
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.6);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal {
  background: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  width: 100%;
  max-width: 500px;
}

.modal-input {
  width: 100%;
  padding: 12px;
  margin-bottom: 15px;
  border: 2px solid #88c6d3;
  border-radius: 8px;
}

.confirm-button,
.cancel-button {
  padding: 10px 15px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 1rem;
  transition: transform 0.2s, background-color 0.3s;
}

.confirm-button {
  background: linear-gradient(135deg, #4fa3c1, #3d87a1);
  color: white;
}

.confirm-button:hover {
  background: linear-gradient(135deg, #3a8f99, #357885);
  transform: scale(1.05);
}

.cancel-button {
  background: linear-gradient(135deg, #f7c0a4, #f2a98d);
  color: white;
}

.cancel-button:hover {
  background: linear-gradient(135deg, #f0a177, #e88f6e);
  transform: scale(1.05);
}
</style>
