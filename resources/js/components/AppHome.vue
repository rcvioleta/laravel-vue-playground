<template>
  <div>
    <h1>Welcome Home</h1>
    <div v-for="todo in myTodos.data" :key="todo.id">
      <div :style="[todo.status ? {backgroundColor: 'white'} : {backgroundColor: 'red'}]">
        <h1>{{ todo.title }}</h1>
        <p>{{ todo.content }}</p>
      </div>
      <q>Created at: {{ todo.created_at }}</q>
      <select name="status" @change="updateStatus($event, todo.id)" v-model="todo.status">
        <option :value="todo.status ? 1 : 0">Currently {{ todo.status ? 'Active' : 'Inactive' }}</option>
        <option :value="todo.status ? 0 : 1">{{ todo.status ? 'Inactive' : 'Active' }}</option>
      </select>

      <button @click="removeTodo(todo.id)">Remove</button>
      <button @click="editTodo(todo.id)">Edit</button>
    </div>
    <!-- <button @click="addTodo">Click me to add new todo</button> -->

    <div :style="style">
      <div style="text-align: right; cursor: pointer" @click="style.display = 'none'">X</div>
      <div v-for="todo in selectedTodo" :key="todo.id">
        <label style="display: block;" for="todo-title">Title</label>
        <input type="text" id="todo-title" name="todo_title" :value="todo.title">
        <br>
        <label style="display: block;" for="todo-content">Content</label>
        <textarea name="todo_content" id="todo-content" v-model="todo.content"></textarea>
        <br>
        <button>Update Todo</button>
      </div>
    </div>

    <pagination :data="myTodos" @pagination-change-page="getResults"></pagination>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      // myTodos: [],
      myTodos: {},
      selectedTodo: [],
      style: {
        position: "absolute",
        top: "50%",
        left: "50%",
        transform: "translate(-50%, -50%)",
        display: "none",
        backgroundColor: "purple",
        color: "white",
        padding: "2rem",
        borderRadius: "5px"
      }
    };
  },
  created() {
    console.log("[created hook]");
    axios
      .get("/api/todo")
      .then(response => {
        console.log("RESPONSE", response.data);
        // this.myTodos = response.data.data;
        this.myTodos = response.data;
      })
      .catch(err => console.log(err.response));
  },
  beforeUpdate() {
    console.log("[beforeUpdate hook]", this.myTodos.data);
  },
  methods: {
    getResults(page = 1) {
      axios.get("/api/todo?page=" + page).then(response => {
        this.myTodos = response.data;
      });
    },
    editTodo(todo_id) {
      console.log(todo_id);
      this.selectedTodo = this.myTodos.data.filter(todo => todo.id === todo_id);
      this.style.display = "block";
    },
    removeTodo(todo_id) {
      axios
        .delete("api/todo/" + todo_id)
        .then(response => {
          this.myTodos.data = this.myTodos.data.filter(
            todo => todo.id !== todo_id
          );
          swal("Congrats!", "Successfully delete todo", "success");
        })
        .catch(err => console.log(err.response));
    },
    updateStatus(e, todo_id) {
      axios
        .post("api/todo/updateStatus", {
          todo_id,
          status: e.target.value
        })
        .then(response => {
          console.log("[updateStatus]", response);
          swal("Success!", "Successfully updated status", "success");
        })
        .catch(err => console.log("err", err.response));
    },
    addTodo() {
      axios
        .post("/api/todo", {
          title: "",
          content: "Kenneth walay duka"
        })
        .then(result => {
          console.log("[addTodo]", result);
          this.myTodos.unshift(result.data);
        })
        .catch(err => console.log(err.response.data));
    }
  }
};
</script>

<style>
</style>
