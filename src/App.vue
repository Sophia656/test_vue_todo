<template>
    <div class="app">
        <form class="input__wrapper">
          <input v-focus :value="text" @input="text = $event.target.value" id="search" />
          <button :disabled="text.length === 0" @click="createTodo">Добавить задачу</button>
        </form>

        <my-select
        v-model="selectedSort"
        :options="sortOptions"
        />

        <div class="todos__wrapper" v-if="innerData.activeFilter == 'active'">
          <div class="todos" v-for="todo in activeTodos" :key="todo.id" >
            <span>{{ todo.index }}. {{ todo.text }}</span>
            <button @click="removeTodo(todo)">Удалить из активных</button>
          </div>
        </div>

        <div class="todos__wrapper" v-if="innerData.activeFilter == 'all'">
          <div class="todos" v-for="todo in innerData.zadachi" :key="todo.id">
            <span>{{ todo.index }}. {{ todo.text }}</span>
            <button v-if="todo.active === false" @click="removeTodo(todo)">Удалить навсегда</button>
            <button v-else @click="removeTodo(todo)">Удалить из активных</button>
          </div>
        </div>

        <div class="todos__wrapper" v-if="innerData.activeFilter == 'completed'">
          <div class="todos" v-for="todo in complitedTodos" :key="todo.id">
            <span>{{ todo.index }}. {{ todo.text }}</span>
            <button @click="removeTodo(todo)">Удалить навсегда</button>
          </div>
        </div>
    </div>
</template>

<script>
import axios from 'axios';
import MySelect from './UI/MySelect.vue';
export default {
  components: { MySelect },
  data() {
    return {
      innerData: {
        zadachi: [],
        // для первоначального вывода на экран --> выводим все задачи
        activeFilter: "all"
      },
      sortOptions: [
          {value: 'all', name: 'Все'},
          {value: 'active', name: 'Активные'},
          {value: 'completed', name: 'Завершенные'}
      ],
      selectedSort: '',
      id: Number,
      text: '',
      active: Boolean,
      activeTodos: [],
      complitedTodos: [],
      index: 1,
      isActive: false
    }
  },
  methods: {
    // делаем запрос на сервер
    async fetchTodo() {
      const responce = await axios.get('https://my-json-server.typicode.com/falk20/demo/todos')
      this.innerData.zadachi = responce.data
      // если имеются те, у которых поле active === true --> пушим в "активные"
      this.innerData.zadachi.map(todo => {
        this.index = todo.id
        Object.assign(todo, {index: this.index})
        if (todo.active === true) {
          this.activeTodos.push(todo)
        } else {
          // у которых active === false --> пушатся в "завершенные"
          this.complitedTodos.push(todo)
        }
      })
    },
    // по клику на ту или иную кнопку --> выводим разные массивы
    handleClickToActive() {
      this.innerData.activeFilter = 'active'
      this.isActive = true
    },
    handleClickToAll() {
      this.innerData.activeFilter = 'all'
      this.isActive = true
      
    },
    handleClickToCompleted() {
      this.innerData.activeFilter = 'completed'
      this.isActive = true
    },
    // создаем новую задачу
    createTodo() {
      const newTodo = {
        id: Date.now(),
        text: this.text,
        active: true ,
        index: ++this.index
      }
      // и пушим ее в основной массив
      this.innerData.zadachi.push(newTodo)
      // а также в массив активных
      this.activeTodos.push(newTodo)
      // сразу очищаем поле инпута
      this.text = ''
    },
    // при удалении(добавления в "завершенные"):
    removeTodo(todo) {
      // проверяем, если у задачи поле active === true, то:
      if (todo.active === true) {
        // удаляем из "активных"
        this.activeTodos = this.activeTodos.filter(p => p.id !== todo.id)
        // делаем active задачи false
        todo.active = false
        // и пушим в массив "завершенных"
        this.complitedTodos.push(todo)
      } else {
        // иначе удаляем ее "c концами" из "всех" и "завершенных"
        this.complitedTodos = this.complitedTodos.filter(p => p.id !== todo.id)
        this.innerData.zadachi = this.innerData.zadachi.filter(p => p.id !== todo.id)
      }
    },
  },
  // чтобы отрисовка задач происходила при первом же открытии
  mounted() {
    this.fetchTodo()
  },
  // вотчер для отрисовки списков задач по выбранному из списка параметру
  watch: {
    selectedSort(newValue) {
      if(newValue === 'all') {
        this.innerData.activeFilter = 'all'
      } else if (newValue === 'active') {
        this.innerData.activeFilter = 'active'
      } else if (newValue === 'completed') {
        this.innerData.activeFilter = 'completed'
      }
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  letter-spacing: 0.1vw;
}
input, textarea, button {
  outline: none;
  border: none;
  background: transparent;
  border-bottom: solid 1px #153c63;
  background-color: rgb(197, 203, 211);
  margin: 2vh 0;
  border: 1px solid lightslategrey;
  width: 20vw;
  height: 4vh;
  text-align: center;
}
html {
    font-size: 16px;
}
.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 100vw;
  height: 100vh;
  background-color: rgb(238, 237, 237);
}
.input__wrapper {
  display: flex;
}
.todos__wrapper {
  display: flex;
  flex-direction: column;
  width: 40%;
}
.todos {
  background-color: rgb(219, 216, 214);
  border: 1px solid lightslategrey;
  margin-bottom: 1vh;
  display: flex;
  justify-content: space-between;
  padding: 0 1vw;
  align-items: center;
}
.active {
  background: lightblue;
}
</style>