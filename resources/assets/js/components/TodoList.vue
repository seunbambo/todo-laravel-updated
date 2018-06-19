<template>
 <div>
     <input type="text" class="todo-input" placeholder="Your todo" v-model="newTodo" @keyup.enter="addTodo"> 
     <div v-for="(todo, index) in todosFiltered" :key="todo.id" class="todo-item">
         <div class="todo-item-left">
             <input type="checkbox" name="check-box" v-model="todo.completed">
             <div v-if="!todo.editing" @dblclick="editTodo(todo)" class="todo-item-label" :class="{ completed : todo.completed }">{{ todo.title }}</div>
            <input v-else class="todo-item-edit" type="text" v-model="todo.title" @blur="doneeEdit(todo)" @keyup.enter="doneEdit(todo)" v-focus>
         </div>
         <div class="remove-item" @click="removeTodo(index)">
            &times;
         </div>
     </div>

     <div class="extra-container">
         <div><label><input type="checkbox" :checked="!anyRemaining" @change="checkAllTodos">Check All</label></div>
         <div>{{ remaining }} items left</div>
     </div>
     <div class="extra-container">
         <div>
            <button :class="{ active : filter == 'all'}" @click="filter = 'all'">All</button>
            <button :class="{ active : filter == 'active'}" @click="filter = 'active'">Active</button>
            <button :class="{ active : filter == 'completed'}" @click="filter = 'completed'">Completed</button>
         </div>

         <div>
             <button v-if="showClearCompletedButton" @click="clearCompleted">Clear Completed</button>
         </div>
     </div>
</div>
</template>

<script>
import axios from 'axios'

axios.defaults.baseURL = 'http://127.0.0.1:8000/api'

let self = this;

export default {
  name: 'todo-list',
  data () {
    return {
      newTodo: '',
      idForTodo: 3,
      beforeEditCache: '',
      filter: 'all',
      todos: [
         /* {
              'id': 1,
              'title': 'Bathing',
              'completed': false,
              'editing': false,
          },
          {
              'id': 2,
              'title': 'Dinning',
              'completed': false,
              'editing': false,
          },
          */
      ]
    }
  },
  created() {
    axios.get('/todos')
    .then(response => {
      // JSON responses are automatically parsed.
      this.todos = response.data
    })
    .catch(e => {
      this.errors.push(e)
    })

  },
  retrieveTodos(context, todos) {
      this.todos = todos
  },
  computed: {
      remaining() {
          return this.todos.filter(todo => !todo.completed).length
      },
      anyRemaining() {
          return this.remaining != 0
      },
      todosFiltered () {
          if(this.filter == 'all') {
              return this.todos
          } else if (this.filter == 'active') {
              return this.todos.filter(todo => !todo.completed)
          } else if (this.filter == 'completed') {
              return this.todos.filter(todo => todo.completed)
          }
          return this.todos
      },
      showClearCompletedButton() {
          return this.todos.filter(todo => todo.completed).length > 0
      }
  },
  directives: {
  focus: {
    inserted: function (el) {
      el.focus()
    }
  }
},
  methods: {
      addTodo(todo){
          
        axios.post('/todos', {
            title: 'title',
            completed: false,
        })
        
          if (this.newTodo.trim().length == 0) {
              return  
          }
          this.todos.push({
              id: this.idForTodo,
              title: this.newTodo,
              completed: false,
          })

          this.newTodo = ''
          this.idForTodo++
      },
      editTodo(todo) {
          axios.post('/todos/' + id, {
            title: todo.title,
            completed: false,
        })
        .then(function (response) {
            console.log(response);
        })
        .catch(function (error) {
            console.log(error);
        })
          this.beforeEditCache = todo.title
          todo.editing = true
      },
      doneEdit(todo) {
           if (todo.title.trim() == '') {
              todo.title = this.beforeEditCache  
          }
          todo.editing= false
      },
      removeTodo(context, id) {
          axios.delete('/todos/' + id)
        .then(response => {
            context.commit('removeTodo', id)
        })
        .catch(error => {
            console.log(error)
        })

         // this.todos.splice(index, 1)
      },
      checkAllTodos() {
        this.todos.forEach((todo) => todo.completed = event.target.checked)
      },
      clearCompleted() {
          this.todos = this.todos.filter(todo => !todo.completed)
      }
  }
}
</script>

<style lang="scss">
.todo-input {
    width: 100%;
    padding: 10px 18px;
    font-size: 18px;
    margin-bottom: 16px;

    a:focus {
        outline: 0;
    }
}

.todo-item {
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    justify-content: space-between;
}

.remove-item {
    cursor: pointer;
    margin-left: 14px;
    b:hover {
        color: green;
    }
}

.todo-item-left {
    display: flex;
    align-items: center;
}

.todo-item-label {
    padding: 10px;
    border: 1px solid white;
    margin-left: 12px;
}

.todo-item-edit {
    font-size: 24px;
    color: #2c3e50;
    margin-left: 12px;
    width: 100%;
    padding: 10px;
    border: 1px solid #ccc;
    font-family:'Avenir', Helvetica, Arial, sans-serif;
}
.completed {
    text-decoration: line-through;
    color: grey;
}

.extra-container {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    border-top: 1px solid lightgrey;
    padding-top: 14px;
    margin-bottom: 14px;

    on:hover {
        background: lightgreen;
    }
    on:focus {
        outline: none;
    }
}

.active {
    background: lightgreen;
}

button {
    font-size: 14px;
    background-color: white;
}

</style>
