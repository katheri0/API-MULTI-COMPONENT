<script setup>
// ##_______________________________________________________________________________________##
// ##____________________________________imports___________________________________________##
import { ref, onMounted } from 'vue';
import axios from 'axios';
// ##_______________________________________________________________________________________##
// ##____________________________________Status logic______________________________________##
const status = ref('active');
const toggleStatus = () => {
    status.value = status.value === 'active'
        ? 'pending'
        : status.value === 'pending'
            ? 'inactive'
            : 'active';
};
// ##_______________________________________________________________________________________##
// ##____________________________________Task logic________________________________________##
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
// ##_______________________________________________________________________________________##
// ##__________________________________CURD Logic___________________________________________##

const users = ref([]);              // All users
const selectedUserId = ref(null);   // Currently selected user for update/delete
const selectedUser = ref(null);     // Currently selected user's data

// =========================
// Fetch all users
// =========================
const fetchUsers = async (id) => {
    try {
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/users');
        users.value = data;
    } catch (error) {
        console.error('Error fetching users:', error.message);
    }
};
// =========================
// Fetch first user
// =========================
const user = ref({});
const Fetch1user = async () => {
    try {
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/users/1');
        user.value = data;
    }
    catch (error) {
        console.error('Error fetching user:', error.message);
    }
};

// =========================
// Fetch single user
// =========================
const fetchUserById = async () => {
    try {
        const { data } = await axios.get('https://jsonplaceholder.typicode.com/users/1');
        selectedUser.value = data;
        selectedUserId.value = id;
    } catch (error) {
        console.error('Error fetching user:', error.message);
    }
};

// =========================
// Create User (POST)
// =========================


const newUser = ref({ name: '', email: '' });
const postResult = ref(null);

const createUser = async () => {
    try {
        const { data } = await axios.post('https://jsonplaceholder.typicode.com/users', newUser.value);
        postResult.value = data;

        // Update local list
        users.value.push({ ...data, id: users.value.length + 1 });
        newUser.value = { name: '', email: '' };
    } catch (err) {
        postError.value = 'Error posting data';
        console.error(err);
    }
};

// =========================
// Update User (PUT) - full replacement
// =========================

const updatedUser = ref({ name: '', email: '' });
const putResult = ref(null);

const updateWithPut = async () => {
    if (!selectedUserId.value) return alert('No user selected!');
    try {
        const { data } = await axios.put('https://jsonplaceholder.typicode.com/users/1', updatedUser.value);
        putResult.value = data;

        // Update locally
        const idx = users.value.findIndex(u => u.id === selectedUserId.value);
        if (idx !== -1) users.value[idx] = { ...data };
    } catch (err) {
        console.error('PUT error:', err);
    }
};

// =========================
// Update User (PATCH) - partial update
// =========================

const patchUser = ref({ name: '' });
const patchResult = ref(null);

const updateWithPatch = async () => {
    if (!selectedUserId.value) return alert('No user selected!');
    try {
        const { data } = await axios.patch('https://jsonplaceholder.typicode.com/users/1', patchUser.value);
        patchResult.value = data;

        // Update locally
        const idx = users.value.findIndex(u => u.id === selectedUserId.value);
        if (idx !== -1) users.value[idx] = { ...users.value[idx], ...data };
    } catch (err) {
        console.error('PATCH error:', err);
    }
};


// =========================
// Delete User (DELETE)
// =========================
const deleteResult = ref(null);

const deleteUser = async (id) => {
    try {
        await axios.delete('https://jsonplaceholder.typicode.com/users/1');
        deleteResult.value = `User ${id} deleted`;
        // Remove from local list
        users.value = users.value.filter(u => u.id !== id);
    } catch (err) {
        console.error('DELETE error:', err);
    }
};

// =========================
// Select User for Update
// =========================
const selectUserForEdit = (user) => {
    selectedUserId.value = user.id;
    updatedUser.value = { name: user.name, email: user.email };
    patchUser.value = { name: user.name };
};
onMounted(Fetch1user);

</script>

<template>
    <div class="container">
        <section class="card">
            <h3>User</h3>
            <h1>{{ user.name }}</h1>
            <h2>{{ user.email }}</h2>
            <p class="status">Status: <span :class="status">{{ status }}</span></p>
            <button @click="toggleStatus" class="btn btn-info">Change Status</button>
        </section>
        <section class="card">

            <form @submit.prevent="addTask" class="task-form">
                <label for="newTask">Add Task:</label>
                <input v-model="newTask" id="newTask" placeholder="Enter a task" />
                <button type="submit" class="btn btn-primary">Add</button>
            </form>

            <h3>Tasks</h3>
            <ul class="task-list">
                <li v-for="(task, index) in tasks" :key="task">
                    <span>{{ task }}</span>
                    <button @click="deleteTask(index)" class="btn btn-danger">✕</button>
                </li>
            </ul>
        </section>
        <!-- _________________________________________________ -->
        <!-- User List -->
        <section class="card">
            <h3>Users</h3>
            <ul> 
                <li class="d-flex  text-center align-items-center m-1 p-1 rounded-3 justify-content-between li-m" v-for="user in users" :key="user.id">
                    {{ user.name }} ({{ user.email }})
                    <div>
                        <button class="btn btn-primary m-1" @click="selectUserForEdit(user)">Edit</button>
                        <button class="btn btn-danger m-1" @click="deleteUser(user.id)">Delete</button>
                    </div>

                </li>
            </ul>
        </section>
        
        <!-- ________________________________________________ -->
        <!-- Create User -->
        <form @submit.prevent="createUser">
            <section class="card">
                <h3>Create New User</h3>
                <input class="m-2" v-model="newUser.name" placeholder="Name" type="text" required />
                <input class="m-2" v-model="newUser.email" placeholder="Email" type="email" required />
                <button class="btn btn-primary m-2" type="submit">Create</button>
                <div v-if="postResult">Created ID: {{ postResult.id }}</div>
            </section>
        </form>
        <!-- _________________________________________________ -->
        <!-- Update User (PUT) -->
        <form  @submit.prevent="updateWithPut">
            <section class="card" v-if="selectedUserId">
                <h3>Update User (PUT)</h3>
                <input class="m-1" v-model="updatedUser.name" placeholder="Name"  type="text" required/>
                <input class="m-1" v-model="updatedUser.email" placeholder="Email" type="email" required/>
                <button class="btn m-1" type="submit">Update (PUT)</button>
                <div v-if="putResult">Updated: {{ putResult.name }}</div>
            </section>
        </form>
        <!-- _________________________________________________ -->
        <!-- Update User (PATCH) -->
        <form @submit.prevent="updateWithPatch">
            <section class="card" v-if="selectedUserId">
                <h3>Update User Name Only (PATCH)</h3>
                <input class="m-1" v-model="patchUser.name" placeholder="Name" />
                <button class="btn m-1" type="submit">Update (PATCH)</button>
                <div v-if="patchResult">Updated Name: {{ patchResult.name }}</div>
            </section>
        </form>

        <!-- Delete Result -->
        <section class="card">
            <div v-if="deleteResult">{{ deleteResult }}</div>
        </section>
    </div>
</template>

<style scoped>
.li-m
{
    background-color: #abc5de;
}
.container {
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

/* .btn {
    padding: 3px 6px;
    background-color: #007bff;
    color: #fff;
    border: none;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.2s ease;
} */

/* .btn:hover {
    background-color: #0056b3;
} */

/* .delete-btn {
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
} */
</style>
