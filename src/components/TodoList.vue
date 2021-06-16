<template>
  <div class="todo">
    <h1>
      おはよう{{ title }}
    </h1>
    <form @submit.prevent="addTodo()">
      <input type="text" class="inputTodo" v-model="newTodo" />
      <input type="submit" value="ADD" class="button" />
    </form>
    <div class="todoListSection">
      <select
        name="Sorting"
        class="sorting"
        v-model="sortSelected"
        @change="sortList()"
      >
        <option value="sortId">追加順</option>
        <option value="sortIsDone">未完順</option>
      </select>
      <ul>
        <li v-for="todo in displayTodo" :key="todo.id">
          <div class="view" v-if="!todo.isEditing">
            <input type="checkbox" v-model="todo.isDone" class="check" />
            <label :class="{ done: todo.isDone }" @dblclick="editTodo(todo)">
              {{ todo.title }}
            </label>
            <button @click="deleteTodo(todo.id)" class="delete">×</button>
          </div>
          <input
            type="text"
            id="edit"
            v-model="todo.title"
            v-else
            @blur="doneEdit(todo, todo.id)"
            @keyup.enter="doneEdit(todo, todo.id)"
          />
        </li>
        <li v-show="!todos.length">Nothing to do !</li>
      </ul>
    </div>
    <div class="infoSection">
      <p class="info">({{ remaining.length }}/{{ todos.length }}) items left</p>
      <button class="purgeBtn" @click="purge()">Purge</button>
    </div>
    <ul class="filter">
      <li>
        <button
          @click="filterAll()"
          class="filterBtn"
          id="all"
          :class="{ selected: visibility === 'all' }"
        >
          All
        </button>
      </li>
      <li>
        <button
          @click="filterWorking()"
          class="filterBtn"
          id="working"
          :class="{ selected: visibility === 'working' }"
        >
          Working
        </button>
      </li>
      <li>
        <button
          @click="filterCompleted()"
          class="filterBtn"
          id="completed"
          :class="{ selected: visibility === 'completed' }"
        >
          Completed
        </button>
      </li>
    </ul>
  </div>
</template>

<script>
export default {
  name: "TodoList",
  data() {
    return {
      id: 0,
      title: "TodoList",
      newTodo: "",
      todos: [],
      visibility: "all",
      filters: [],
      sortSelected: "sortId",
    };
  },
  watch: {
    // これはtodosの中身までは監視してくれないくれない（深い階層が見れない）
    // todos: function () {
    //   localStorage.setItem("todos", JSON.stringify(this.todos));
    //   alert("Data saved");
    // },
    todos: {
      handler() {
        localStorage.setItem("todos", JSON.stringify(this.todos));
      },
      deep: true,
    },
  },
  mounted() {
    this.todos = JSON.parse(localStorage.getItem("todos")) || [];
  },
  methods: {
    addTodo() {
      if (this.newTodo === "") return;
      let item = {
        id: this.id++,
        title: this.newTodo,
        isDone: false,
        isEditing: false,
      };
      this.todos.push(item);
      this.newTodo = "";
    },
    deleteTodo(deleteId) {
      // this.todos.splice(index, 1);
      this.todos = this.todos.filter((todo) => deleteId !== todo.id);
    },
    editTodo(todo) {
      todo.isEditing = true;
      this.$nextTick(() => document.getElementById("edit").focus());
    },
    doneEdit(todo, id) {
      // event.stopImmediatePropagation();
      todo.title = todo.title.trim();
      if (!todo.title) {
        this.deleteTodo(id);
      }
      todo.isEditing = false;
    },
    purge() {
      if (!window.confirm("完了したものを削除しますか?")) {
        return;
      }
      this.todos = this.remaining;
    },
    filterAll() {
      this.filters = this.todos;
      this.visibility = "all";
    },
    filterWorking() {
      this.filters = this.remaining;
      this.visibility = "working";
    },
    filterCompleted() {
      this.filters = this.completed;
      this.visibility = "completed";
    },
    sortId() {
      this.displayTodo.sort(function (a, b) {
        return a.id - b.id;
      });
    },
    sortIsDone() {
      this.displayTodo.sort(function (a, b) {
        if (a.isDone && !b.isDone) {
          return 1;
        } else if (a.isDone === b.isDone) {
          return a.id - b.id;
        } else {
          return -1;
        }
      });
    },
    sortList() {
      if (this.sortSelected === "sortId") {
        this.sortId();
      } else if (this.sortSelected === "sortIsDone") {
        this.sortIsDone();
      }
    },
  },
  computed: {
    remaining() {
      return this.todos.filter(function (todo) {
        return !todo.isDone;
      });
    },
    completed() {
      return this.todos.filter(function (todo) {
        return todo.isDone;
      });
    },
    displayTodo() {
      return this.visibility !== "all" ? this.filters : this.todos;
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
button {
  cursor: pointer;
}
.todoListSection {
  position: relative;
  width: 70%;
  margin: 30px auto;
}
ul {
  list-style-type: none;
  padding: 20px 0 0 0;
  text-align: left;
}
li {
  position: relative;
  display: block;
  margin: 6px 10px;
  padding: 6px;
  border-bottom: 1px solid #ddd;
}
li:last-child {
  border: none;
}
.inputTodo {
  padding: 3px 6px;
  line-height: 1.5;
  font-size: 14px;
}
.button {
  background-color: #2c3e50;
  color: #fff;
  letter-spacing: 1px;
  padding: 5px 12px;
  margin-left: 12px;
  line-height: 1.5;
  font-size: 14px;
  border: none;
  cursor: pointer;
}
.delete {
  position: absolute;
  top: 5px;
  right: 0;
  margin-left: 10px;
  vertical-align: middle;
  font-size: 30px;
  background: none;
  border: none;
}
.check {
  position: absolute;
  top: 13px;
  left: 0;
}
.view label {
  display: block;
  width: calc(100% - 20px);
  padding: 6px 0;
  padding-left: 20px;
}
ul li .view > label.done {
  text-decoration: line-through;
  color: #bbb;
}
#edit {
  margin: 6px 10px;
  padding: 6px;
  margin-left: 14px;
  width: calc(100% - 20px);
}
.infoSection {
  display: flex;
  justify-content: space-between;
  margin: 0 auto;
  width: 70%;
}
.info {
  margin: 0;
  padding: 0;
  color: #aaa;
  font-size: 16px;
  text-align: left;
}
.purgeBtn {
  padding: 6px 12px;
  border: none;
  color: #fff;
  background: #42b983;
  border-radius: 3px;
  font-size: 14px;
}
.filter {
  display: flex;
  justify-content: center;
  margin: 10px auto;
  text-align: center;
}
.filter li {
  margin: 0 6px;
  padding: 0;
  border: none;
}
.filterBtn {
  padding: 3px 6px;
  border: none;
  color: #999;
  background: none;
}
.selected {
  border: 1px solid #42b983;
  border-radius: 3px;
  color: #666;
}
.sorting {
  position: absolute;
  top: 0;
  right: 0;
  padding: 3px 5px;
  font-size: 12px;
}
.sorting span {
  color: #42b983;
}
.sorting button {
  background: none;
  border: none;
  color: #42b983;
}
</style>

