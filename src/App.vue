<template>
  <div id="app">
    <to-do-list></to-do-list>
  </div>
</template>

<script>
import ToDoList from './components/ToDoList.vue'
export default {
  name: "App",
  components:{
    ToDoList
  }
};
</script>
