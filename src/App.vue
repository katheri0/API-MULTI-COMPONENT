<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';

// Status logic
const status = ref('active');
const toggleStatus = () => {
  status.value = status.value === 'active'
    ? 'pending'
    : status.value === 'pending'
      ? 'inactive'
      : 'active';
};

// Task logic
const tasks = ref([]);
const newTask = ref('');

const addTask = () => {
  if (newTask.value.trim()) {
    tasks.value.push(newTask.value.trim());
    newTask.value = '';
  }
};

const deleteTask = (index) => {
  tasks.value.splice(index, 1);
};

onMounted(async () => {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/todos');
    const data = await response.json();
    tasks.value = data.slice(0, 5).map(task => task.title); // Only 5 tasks
  } catch (error) {
    console.error('Error fetching tasks');
  }
});

// Fetch all users
const users = ref([]);
onMounted(async () => {
  try {
    const { data } = await axios.get('https://jsonplaceholder.typicode.com/users');
    users.value = data;
  } catch (error) {
    console.error('Error fetching users:', error.message);
  }
});

// Fetch single user
const user = ref({});
onMounted(async () => {
  try {
    const { data } = await axios.get('https://jsonplaceholder.typicode.com/users/1');
    user.value = data;
  } catch (error) {
    console.error('Error fetching user:', error.message);
  }
});

// Create user (POST)
const newUser = ref({
  name: '',
  email: ''
});

const addNewUser = () => {
  if (newUser.value.trim()) {
    user.value.push(newUser.value.trim());
    newUser.value = '';
  }
};
const postResult = ref(null);
const postError = ref(null);

const createUser = async () => {
  try {
    const { data } = await axios.post('https://jsonplaceholder.typicode.com/users', newUser.value);
    postResult.value = data;
  } catch (err) {
    postError.value = 'Error posting data';
    console.error(err);
  }
};

// Update with PUT
const updatedUser = ref({
  name: 'Jane Smith',
  email: 'jane@example.com'
});
const putResult = ref(null);

const updateWithPut = async () => {
  try {
    const { data } = await axios.put('https://jsonplaceholder.typicode.com/users/1', updatedUser.value);
    putResult.value = data;
  } catch (err) {
    console.error('PUT error:', err);
  }
};

// Update with PATCH
const patchUser = ref({
  name: 'Jane Updated'
});
const patchResult = ref(null);

const updateWithPatch = async () => {
  try {
    const { data } = await axios.patch('https://jsonplaceholder.typicode.com/users/1', patchUser.value);
    patchResult.value = data;
  } catch (err) {
    console.error('PATCH error:', err);
  }
};

const DeleteUser = ref({
  name: 'Jane'
});
const deleteResult= ref(null);
const dElete = async () => {
  try {
    const { data } = await axios.delete('https://jsonplaceholder.typicode.com/users/1');
    deleteResult.value = data;
  } catch (err) {
    console.error('DELETE error:', err);
  }
};
</script>

<template>
  <div class="container">
    <section class="card">
      <h1>{{ user.name }}</h1>
      <h2>{{ user.email }}</h2>
      <p class="status">Status: <span :class="status">{{ status }}</span></p>
      <button @click="toggleStatus" class="btn">Change Status</button>
    </section>

    <section class="card">
      <form @submit.prevent="addTask" class="task-form">
        <label for="newTask">Add Task:</label>
        <input v-model="newTask" id="newTask" placeholder="Enter a task" />
        <button type="submit" class="btn">Add</button>
      </form>

      <h3>Tasks</h3>
      <ul class="task-list">
        <li v-for="(task, index) in tasks" :key="task">
          <span>{{ task }}</span>
          <button @click="deleteTask(index)" class="delete-btn">✕</button>
        </li>
      </ul>
    </section>

    <section class="card">
      <h3>Users List</h3>
      <ul>
        <li v-for="user in users" :key="user.id">
          {{ user.name }} ({{ user.email }})
        </li>
      </ul>
    </section>

    <section class="card">
      <h3>POST New User</h3>
      <input v-model="newUser.name" placeholder="Enter name" />
      <input v-model="newUser.email" placeholder="Enter email" />
      <button @click="createUser" class="btn">Create User</button>
      <div v-if="postResult" class="info">Created User ID: {{ postResult.id }}</div>
      <div v-if="postError" class="error">{{ postError }}</div>
    </section>


    <section class="card">
      <h3>PUT Update</h3>
      <input v-model="updatedUser.name" placeholder="Enter name" />
      <input v-model="updatedUser.email" placeholder="Enter email" />
      <button @click="updateWithPut" class="btn">Update with PUT</button>
      <div v-if="putResult" class="info">
        Updated: {{ putResult.name }} — {{ putResult.email }}
      </div>
    </section>

    <section class="card">
      <h3>PATCH Update</h3>
      <input v-model="patchUser.name" placeholder="Enter name" />
      <button @click="updateWithPatch" class="btn">Update with PATCH</button>
      <div v-if="patchResult" class="info">Updated Name: {{ patchResult.name }}</div>
    </section>

    <section class="card">
      <h3>DELETE </h3>
      <button @click="dElete" type="button" class=" delete-btn">delete</button>
      <div v-if="deleteResult" class="info">Deleted Name: {{ DeleteUser.name }}</div>
    </section>
  </div>
</template>

<style scoped>
.container {
  max-width: 720px;
  margin: 32px auto;
  padding: 16px;
  font-family: 'Segoe UI', sans-serif;
}

.card {
  background-color: #818181;
  padding: 24px;
  margin-bottom: 24px;
  border-radius: 12px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.05);
}

h1 {
  font-size: 26px;
  color: #333;
  margin-bottom: 0.5rem;
}

h2 {
  font-size: 18px;
  color: #666;
  margin-bottom: 1rem;
}

h3 {
  margin-bottom: 16px;
  color: #333;
  font-weight: 600;
}

.status span {
  font-weight: bold;
  text-transform: capitalize;
}

.status .active {
  color: green;
}

.status .pending {
  color: orange;
}

.status .inactive {
  color: red;
}

.task-form {
  display: flex;
  gap: 8px;
  align-items: center;
  margin-bottom: 16px;
}

input {
  padding: 8px;
  flex: 1;
  border: 1px solid #ccc;
  border-radius: 6px;
}

ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.task-list li {
  display: flex;
  justify-content: space-between;
  padding: 7px 0;
  border-bottom: 1px solid #eee;
}

.btn {
  padding: 3px 6px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.2s ease;
}

.btn:hover {
  background-color: #0056b3;
}

.delete-btn {
  background: none;
  background-color: #ff0000;
  border-radius: 6px;
  color: #fff;
  border: none;
  font-size: 10px;
  cursor: pointer;
  padding: 9px;
}

.info {
  margin-top: 8px;
  background-color: #071b23;
  padding: 8px;
  border-left: 4px solid #1890ff;
}

.error {
  margin-top: 8px;
  color: red;
  font-weight: bold;
}
</style>
