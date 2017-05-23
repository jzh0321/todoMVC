<template>
  <section class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input class="new-todo" placeholder="What needs to be done?" autofocus v-model='newTodo' @keyup.enter="addTodo">
    </header>
    <!-- This section should be hidden by default and shown when there are todos -->
    <section class="main" v-show="todos.length">
      <input class="toggle-all" type="checkbox" id="toggle-all" v-model="allDone">
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <!-- These are here just to show the structure of the list items -->
        <!-- List items should get the class `editing` when editing and `completed` when marked as completed -->
        <li v-for="(todo, index) in filterTodos" :class="{completed: todo.completed, editing: todo === editedTodo}" :key="todo.id">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label for="toggle" @dblclick="editTodo(todo)">{{todo.title}}</label>
            <button class="destroy" @click='removeTodo(todo)'></button>
          </div>
          <input class="edit" v-model.trim="todo.title" @keyup.esc="cancelEdit(todo)" v-todo-focus="todo === editedTodo" @blur="doneEdit(todo)" @keyup.enter="doneEdit(todo)">
        </li>
      </ul>
    </section>
    <!-- This footer should hidden by default and shown when there are todos -->
    <footer class="footer"  v-show="todos.length">
      <!-- This should be `0 items left` by default -->
      <span class="todo-count">
        <strong>{{remianing}}</strong> {{remianing | pluralize}} left</span>
      <!-- Remove this if you don't implement routing -->
      <ul class="filters">
        <li>
          <a href="javascript: void(0);" @click="visibility = 'all'" :class="{'selected': visibility === 'all'}">All</a>
        </li>
        <li>
          <a href="javascript: void(0);" @click="visibility = 'active'" :class="{'selected': visibility === 'active'}">Active</a>
        </li>
        <li>
          <a href="javascript: void(0);" @click="visibility = 'completed'" :class="{'selected': visibility === 'completed'}">Completed</a>
        </li>
      </ul>
      <!-- Hidden if no completed items are left ↓ -->
      <button class="clear-completed" @click="clearCompleted">Clear completed</button>
    </footer>
  </section>
  
  <!-- Scripts here. Don't remove ↓ -->
  <!--<script src="node_modules/todomvc-common/base.js"></script>-->
</template>

<style>
@import url('/static/css/base.css');
@import '/static/css/index.css';
</style>

<script>
// localStorage persistence
var STORAGE_KEY = 'todos-vuejs-2.0'
var todoStorage = {
  fetch: function () {
    var todos = JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]')
    todos.forEach(function (todo, index) {
      todo.id = index
    })
    todoStorage.uid = todos.length
    return todos
  },
  save: function (todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}
// 过滤数据
let filters = {
  all: function (todos) {
    return todos
  },
  active: function (todos) {
    return todos.filter((todo) => {
      return !todo.completed
    })
  },
  completed: function (todos) {
    return todos.filter((todo) => {
      return todo.completed
    })
  }
}
export default {
  data () {
    return {
      newTodo: '',
      todos: todoStorage.fetch(), // 本地存储所有计划
      visibility: 'all', // 默认显示所有
      editedTodo: null
    }
  },
  // vue-cli用watch死活报错, this搞不清楚
  watch: {
    // todos: function (todos) {
    //   todoStorage.save(todos)
    // }
    todos: {
      handler: function (todos) {
        todoStorage.save(todos)
      },
      deep: true
    }
  },
  methods: {
    // 添加
    addTodo: function () {
      let val = this.newTodo && this.newTodo.trim()
      if (!val) {
        return
      }
      this.todos.unshift({
        id: todoStorage.uid++,
        title: val,
        completed: false
      })
      this.newTodo = ''
    },
    // 删除
    removeTodo: function (item) {
      let index = this.todos.indexOf(item)
      this.todos.splice(index, 1)
    },
    // 清除全部完成
    clearCompleted: function () {
      this.todos = filters.active(this.todos)
    },
    // 编辑
    editTodo: function (todo) {
      // 保存编辑前数据
      this.beforeEditCache = todo.title
      // 正在编辑的todo
      this.editedTodo = todo
    },
    // 取消编辑
    cancelEdit: function (todo) {
      // 当前编辑的置空
      this.editedTodo = null
      todo.title = this.beforeEditCache
    },
    // 完成编辑
    doneEdit: function (todo) {
      if (!this.editedTodo) {
        return
      }
      this.editedTodo = null
      if (!todo.title) {
        this.removeTodo(todo)
      }
    }
  },
  computed: {
    filterTodos: function () {
      return filters[this.visibility](this.todos)
    },
    remianing: function () {
      return filters.active(this.todos).length
    },
    // 全部完成
    allDone: {
      get: function () {
        return this.remianing === 0
      },
      set: function (value) {
        this.todos.forEach((todo, index) => {
          todo.completed = value
        })
      }
    }
  },
  filters: {
    pluralize: function (n) {
      return n === 1 ? 'item' : 'items'
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
