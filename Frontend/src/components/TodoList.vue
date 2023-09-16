<template>
    <div class="todo-container">
      <div class="todo-list">
        <h1>Todo List</h1>
        <div class="todo-form">
          <input class="todo-input" v-model="newTodo.title" placeholder="Add new todo title" />
          <input class="todo-input" v-model="newTodo.description" placeholder="Add description" />
          <input type="checkbox" v-model="newTodo.completed" /> Completed
          <input type="date" v-model="newTodo.dueDate" />
          <button class="btn btn-add" @click="addTodo">Add Todo</button>
        </div>
      </div>
      <div class="todo-items">
        <table>
          <thead>
            <tr>
              <th>Title</th>
              <th>Description</th>
              <th>Due Date</th>
              <th>Status</th>
              <th>Actions</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="todo in todos" :key="todo._id">
              <td>{{ todo.title }}</td>
              <td>{{ todo.description }}</td>
              <td v-if="todo.dueDate">{{ new Date(todo.dueDate).toLocaleDateString() }}</td>
              <td v-else>Not Set</td>
              <td>
                <span v-if="todo.completed">Completed</span>
                <span v-else>Incomplete</span>
              </td>
              <td>
                <button class="btn btn-update" @click="updateTodo(todo)" title="delete"><i class="fa-regular fa-trash-can"></i></button>
                <button class="btn btn-delete" @click="deleteTodo(todo._id)" title="edit"><i class="fa-solid fa-pen"></i></button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </template>
  
  <script setup>
    import { ref } from 'vue';
    import axios from 'axios';
  
    const todos = ref([]);
    const newTodo = ref({
      title: '',
      description: '',
      completed: false,
      dueDate: null
    });
  
    const fetchTodos = async () => {
      const response = await axios.get('http://localhost:3000/todos');
      todos.value = response.data;
    };
  
    const addTodo = async () => {
      const response = await axios.post('http://localhost:3000/todos_create', newTodo.value);
      todos.value.push(response.data);
      newTodo.value = {
        title: '',
        description: '',
        completed: false,
        dueDate: null
      };
    };
  
    const deleteTodo = async (id) => {
      await axios.delete(`http://localhost:3000/todos/${id}`);
      todos.value = todos.value.filter(todo => todo._id !== id);
    };
  
    const updateTodo = async (todo) => {
      // Prompt the user to update each field
      const newTitle = prompt('Update todo title:', todo.title);
      const newDescription = prompt('Update todo description:', todo.description);
      const newDueDate = prompt('Update todo due date:', todo.dueDate ? new Date(todo.dueDate).toLocaleDateString() : '');
      const newCompleted = confirm('Is the todo completed? Click "OK" for Yes and "Cancel" for No.');

      // Create an updated todo object
      const updatedTodo = {
        ...todo,
        title: newTitle !== null ? newTitle : todo.title,
        description: newDescription !== null ? newDescription : todo.description,
        dueDate: newDueDate !== null ? new Date(newDueDate).toISOString() : todo.dueDate,
        completed: newCompleted
      };

      // Send the updated todo to the server
      const response = await axios.put(`http://localhost:3000/todos/${todo._id}`, updatedTodo);

      // Update the local state
      const index = todos.value.findIndex(t => t._id === todo._id);
      todos.value[index] = response.data;
    };
  
    fetchTodos();
  </script>
