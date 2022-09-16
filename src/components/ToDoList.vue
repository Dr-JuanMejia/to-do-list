<template>
  <div class="container">
    <h3 class="text-center mt-4 mb-4">Example - To Do List</h3>
    <div class="row">
      <div class="col-md-4">
        <input
          v-model="description"
          type="text"
          placeholder="Description..."
          class="form-control"
        />
      </div>
      <div class="col-md-2">
        <button @click="submitDescription" class="btn btn-info">Agregar</button>
      </div>
    </div>
    <table class="table table-bordered mt-4">
      <thead>
        <tr>
          <th scope="col">Description</th>
          <th scope="col">Status</th>
          <th scope="col" class="text-center">Editar</th>
          <th scope="col" class="text-center">Eliminar</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(task, index) in tasks" :key="index">
          <td>{{ task.description }}</td>
          <td>{{ task.state }}</td>
          <td>
            <div class="text-center" @click="editDescription(index)">
              <span class="fa fa-pen"></span>
            </div>
          </td>
          <td>
            <div class="text-center" @click="deleteDescription(index)">
              <span class="fa fa-trash"></span>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="panel">
      <div class="panel-heading">Requerimientos:</div>
      <div class="panel-body">
        <ul>
          <li>Muestra todas las tareas de la lista.</li>
          <li>Nos permite ingresar una tarea a nuestra lista.</li>
          <li>Nos permite eliminar una tarea a nuestra lista.</li>
          <li>Nos permite actualizar una tarea a nuestra lista.</li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
import { API, graphqlOperation } from "aws-amplify";
import { listTodos } from "../graphql/queries";
import { createTodo, deleteTodo, updateTodo } from "../graphql/mutations";
export default {
  name: "ToDoList",
  props: {
    msg: String,
  },
  data() {
    return {
      description: "",
      state: "To Do",
      editDescriptionValue: null,
      tasks: [],
    };
  },
  async mounted() {
    this.getListToDo();
  },
  methods: {
    async getListToDo() {
      try {
        const { data } = await API.graphql(graphqlOperation(listTodos));
        this.tasks = data.listTodos.items;
      } catch (error) {
        console.log(error);
      }
    },
    async submitDescription() {
      if (this.description.length === 0) return;

      if (this.editDescriptionValue === null) {
        try {
          const { data } = await API.graphql(
            graphqlOperation(createTodo, {
              input: { description: this.description, state: this.state },
            })
          );
          this.getListToDo();
          console.log(data);
        } catch (error) {
          console.log(error);
        }

        this.tasks.push({
          description: this.description,
          state: "To Do",
        });
      } else {
        this.tasks[this.editDescriptionValue].description = this.description;
        try {
          const { data } = await API.graphql(
            graphqlOperation(updateTodo, {
              input: {
                id: this.tasks[this.editDescriptionValue].id,
                description: this.tasks[this.editDescriptionValue].description,
              },
            })
          );
          this.getListToDo();
          console.log(data);
        } catch (error) {
          console.log(error);
        }

        this.editDescriptionValue = null;
      }

      this.description = "";
    },
    async deleteDescription(index) {
      try {
        const { data } = await API.graphql(
          graphqlOperation(deleteTodo, {
            input: { id: this.tasks[index].id },
          })
        );
        this.getListToDo();
        console.log(data);
      } catch (error) {
        console.log(error);
      }
    },
    editDescription(index) {
      this.description = this.tasks[index].description;
      this.editDescriptionValue = index;
    },
  },
};
</script>

<style scoped></style>
