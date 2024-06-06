<template>
  <v-container>
    <!-- Search and Filter Options -->
    <v-row>
      <v-col cols="12">
        <v-text-field
          v-model="searchQuery"
          label="Search"
          @input="getTodos"
        ></v-text-field>
      </v-col>
      <v-col cols="12">
        <v-checkbox
          v-model="showCompleted"
          label="Show Completed"
          @change="getTodos"
        ></v-checkbox>
      </v-col>
    </v-row>

    <v-row class="mb-3">
      <v-col cols="12">
        <v-form ref="form" lazy-validation class="text-center">
          <v-text-field
            v-model="newTodo.title"
            :counter="32"
            :rules="newTodo.titleRules"
            label="Title"
            required
          ></v-text-field>
          <v-textarea
            v-model="newTodo.description"
            autocomplete="Description"
            label="Description"
          ></v-textarea>
          <v-checkbox
            v-model="newTodo.is_finished"
            label="Is task finished?"
          ></v-checkbox>
          <v-btn color="error" class="mr-4" @click="reset">Clear</v-btn>
          <v-btn
            color="primary"
            class="mr-4"
            :disabled="!newTodo.title"
            @click="saveTodo"
          >
            {{ isEditing ? "Update" : "Add" }}
          </v-btn>
        </v-form>
      </v-col>
    </v-row>
    <v-divider></v-divider>
    <v-row>
      <v-col cols="12" v-if="filteredTodoList.length">
        <h3>List:</h3>
        <v-list>
          <v-list-item-group v-model="selected" color="info" multiple>
            <v-list-item
              v-for="item in filteredTodoList"
              :key="item.id"
              @click="editTodo(item)"
            >
              <v-list-item-icon>
                {{ item.id }}
              </v-list-item-icon>
              <v-list-item-content>
                <v-list-item-title>
                  <strong>{{ item.title }}</strong>
                </v-list-item-title>
                <v-list-item-subtitle>
                  {{ item.description }}
                </v-list-item-subtitle>
              </v-list-item-content>
              <v-list-item-action>
                <v-checkbox
                  :input-value="item.is_finished"
                  color="info"
                ></v-checkbox>
                <v-btn icon @click.stop="deleteTodo(item.id)">
                  <v-icon color="red">mdi-delete</v-icon>
                </v-btn>
              </v-list-item-action>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import axios from "axios";
export default {
  data: () => ({
    selected: [],
    searchQuery: "",
    showCompleted: true, // Ensure this property is declared
    isEditing: false,
    currentTodoId: null,
    newTodo: {
      title: "",
      description: "",
      titleRules: [
        (v) => !!v || "Title is required",
        (v) => (v && v.length <= 32) || "Title must be less than 32 characters",
      ],
      is_finished: false,
      user: 1,
    },
    todoList: [],
    url: "http://localhost:8000/api/todo/",
  }),
  computed: {
    filteredTodoList() {
      return this.todoList.filter(
        (todo) =>
          (this.showCompleted || !todo.is_finished) &&
          todo.title.toLowerCase().includes(this.searchQuery.toLowerCase())
      );
    },
  },
  mounted() {
    this.getTodos();
  },
  methods: {
    getTodos() {
      const params = {
        ordering: "is_finished",
        search: this.searchQuery,
      };
      if (!this.showCompleted) {
        params.is_finished = false;
      }
      axios.get(this.url, { params }).then((response) => {
        this.todoList = response.data;
      });
    },
    reset() {
      this.$refs.form.reset();
      this.isEditing = false;
      this.currentTodoId = null;
    },
    saveTodo() {
      if (this.isEditing) {
        this.updateTodo();
      } else {
        this.addTodo();
      }
    },
    addTodo() {
      axios.post(this.url, this.newTodo).then((response) => {
        this.getTodos();
        this.reset();
      });
    },
    updateTodo() {
      const url = `${this.url}${this.currentTodoId}/`;
      axios.patch(url, this.newTodo).then((response) => {
        this.getTodos();
        this.reset();
      });
    },
    editTodo(item) {
      this.newTodo.title = item.title;
      this.newTodo.description = item.description;
      this.newTodo.is_finished = item.is_finished;
      this.currentTodoId = item.id;
      this.isEditing = true;
    },
    deleteTodo(id) {
      const url = `${this.url}${id}/`;
      axios.delete(url).then((response) => {
        this.getTodos();
      });
    },
    updateStatus(item) {
      item.is_finished = !item.is_finished;
      const url = `${this.url}${item.id}/`;
      const data = {
        is_finished: item.is_finished,
      };
      axios.patch(url, data).then((response) => {
        this.getTodos();
      });
    },
  },
};
</script>
