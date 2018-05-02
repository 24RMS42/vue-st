<template>
  <div class="login-wrapper border border-light">
    <form class="form-signin" @submit.prevent="register">
      <h2 class="form-signin-heading">Register form</h2>
      <div class="alert alert-danger" v-if="error">{{ error }}</div>
      <label for="inputName" class="sr-only">Name</label>
      <input v-model="name"  type="text" id="inputName" class="form-control" placeholder="Name" required autofocus>
      <label for="inputCount" class="sr-only">Count</label>
      <input v-model="count" type="number" id="inputCount" class="form-control" placeholder="Count" required>
      <div>
          <input type="radio" name="radio-1" id="radio-1" value="fridge1" 
                  v-model="fridge">
          <label for="radio-1">fridge1</label>
          <input type="radio" name="radio-2" id="radio-2" value="fridge2" 
                  v-model="fridge">
          <label for="radio-2">fridge2</label>
      </div>
      <button class="btn btn-lg btn-primary btn-block" type="submit">Register</button>
    </form>
    <h2 class="form-signin-heading">My Grocery</h2>
    <section class="main" v-show="todos.length" v-cloak>
      <ul class="todo-list">
        <li v-for="todo in this.todos"
          class="todo"
          :key="todo.id"
          :class="{ editing: todo == editedTodo }">
          <div class="view">
            <button class="btn-decrease" @click="decrease(todo)">consume</button>
            <label @dblclick="editTodo(todo)">{{ todo.name }}</label>
            <label><span>{{ todo.count }}</span></label>
            <label>{{ todo.fridge }}</label>
          </div>
          <!-- <input class="edit" type="text"
            v-model="todo.name"
            v-todo-focus="todo == editedTodo"
            @blur="doneEdit(todo)"
            @keyup.enter="doneEdit(todo)"
            @keyup.esc="cancelEdit(todo)"> -->
        </li>
      </ul>
    </section>
    <button class="btn-decrease" @click="filter()">Movement</button>
    <h2 class="form-signin-heading">Fridge1</h2>
    <section class="main" v-show="this.fridge1.length" v-cloak>
      <ul class="todo-list">
        <li v-for="todo in this.fridge1"
          class="todo"
          :key="todo.id">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.selected">
            <label>{{ todo.name }}</label>
            <label><span>{{ todo.count }}</span></label>
            <!-- <label>{{ todo.fridge }}</label> -->
          </div>
        </li>
      </ul>
    </section>
    <h2 class="form-signin-heading">Fridge2</h2>
    <section class="main" v-show="this.fridge2.length" v-cloak>
      <ul class="todo-list">
        <li v-for="todo in this.fridge2"
          class="todo"
          :key="todo.id">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.selected">
            <label>{{ todo.name }}</label>
            <label><span>{{ todo.count }}</span></label>
            <!-- <label>{{ todo.fridge }}</label> -->
          </div>
        </li>
      </ul>
    </section>
    <button class="btn-decrease" @click="moveOneToTwo()">Move Fridge1 to Fridge2</button>
    <button class="btn-decrease" @click="moveTwoToOne()">Move Fridge2 to Fridge1</button>
    <br>
    <button class="btn-decrease" @click="apply()">Apply</button>
  </div>
</template>

<script>
var STORAGE_KEY = 'freshass'
var todoStorage = {
  fetch: function () {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    todos.forEach(function (todo, index) {
      todo.id = index
      todo.selected = false
    })
    todoStorage.uid = todos.length
    console.log('finding==========', todos)
    return todos
  },
  save: function (todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}
export default {
  name: 'Home',
  data () {
    return {
      name: '',
      count: '',
      error: false,
      fridge: [],
      todos: todoStorage.fetch(),
      editedTodo: null,
      fridge1: [],
      fridge2: [],
      selected: false
    }
  },
  updated () {
  },
  methods: {
    register () {
      if (this.fridge.length === 0) {
        alert('Please select fridge')
        return
      }
      console.log('current todos:', this.todos.length)
      var data = {
        id: todoStorage.uid++,
        name: this.name,
        count: this.count,
        fridge: this.fridge
      }
      const obj = this
      let updated
      this.todos.forEach(function (todo, index) {
        if (todo.name === obj.name && todo.fridge === obj.fridge) { // if same name/fridge
          todo.count = parseInt(todo.count) + parseInt(obj.count)
          updated = true
        }
      })
      if (!updated) { // if different name
        this.todos.push(data)
      }
      console.log('model== data:', this.todos)
      todoStorage.save(this.todos)
    },
    removeTodo (todo) {
      this.todos.splice(this.todos.indexOf(todo), 1)
    },
    editTodo (todo) {
      console.log('editing todo:', todo)
      this.editedTodo = todo
      this.$router.replace('/authors')
    },
    doneEdit (todo) {

    },
    cancelEdit (todo) {

    },
    decrease (todo) {
      console.log('decreasing===', todo)
      todo.count--
      if (todo.count <= 0) {
        this.error = 'Removed one item'
        this.removeTodo(todo)
        todoStorage.save(this.todos)
      } else {
        todoStorage.save(this.todos)
      }
    },
    filter () {
      const obj = this
      obj.fridge1 = []
      obj.fridge2 = []
      this.todos.forEach(function (todo, index) {
        if (todo.fridge === 'fridge1') {
          obj.fridge1.push(todo)
        } else if (todo.fridge === 'fridge2') {
          obj.fridge2.push(todo)
        }
      })
    },
    moveOneToTwo () {
      const obj = this
      var i = this.fridge1.length
      while (i--) {
        var todo = this.fridge1[i]
        if (todo.selected) {
          obj.fridge2.push(todo)
          obj.fridge1.splice(i, 1)
        }
      }
    },
    moveTwoToOne () {
      const obj = this
      var i = this.fridge2.length
      while (i--) {
        var todo = this.fridge2[i]
        if (todo.selected) {
          obj.fridge1.push(todo)
          obj.fridge2.splice(i, 1)
        }
      }
    },
    apply () {
      const obj = this
      obj.todos = []
      this.fridge1.forEach(function (todo, index) {
        todo.fridge = 'fridge1'
        obj.todos.push(todo)
      })
      this.fridge2.forEach(function (todo, index) {
        todo.fridge = 'fridge2'
        obj.todos.push(todo)
      })
      todoStorage.save(obj.todos)
    }
  },
  directives: {
    'todo-focus': function (el, binding) {
      if (binding.value) {
        el.focus()
      }
    }
  }
}
</script>

<style lang="css">
body {
  background: #605B56;
}

.login-wrapper {
  background: #fff;
  width: 70%;
  margin: 12% auto;
}

.form-signin {
  max-width: 330px;
  padding: 10% 15px;
  margin: 0 auto;
}
.form-signin .form-signin-heading,
.form-signin .checkbox {
  margin-bottom: 10px;
}
.form-signin .checkbox {
  font-weight: normal;
}
.form-signin .form-control {
  position: relative;
  height: auto;
  -webkit-box-sizing: border-box;
          box-sizing: border-box;
  padding: 10px;
  font-size: 16px;
}
.form-signin .form-control:focus {
  z-index: 2;
}
.form-signin input[type="email"] {
  margin-bottom: -1px;
  border-bottom-right-radius: 0;
  border-bottom-left-radius: 0;
}
.form-signin input[type="password"] {
  margin-bottom: 10px;
  border-top-left-radius: 0;
  border-top-right-radius: 0;
}
label span { color: red; }
</style>