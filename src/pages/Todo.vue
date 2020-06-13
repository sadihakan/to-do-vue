<template>
  <q-page class="bg-grey-3 column">
    <div class="row q-pa-sm bg-primary">
      <q-input @keyup.enter="addTask" v-model="newTask" filled label="Label" dense bg-color="white" class="col" square placeholder="Add Task">
        <template v-slot:append>
          <q-btn round dense flat icon="add" @click="addTask"/>
        </template>
      </q-input>
    </div>
    <q-list class="bg-white" separator bordered>
      <q-item v-ripple v-for="(task, index) in tasks" :key="task.title"  @click="setTask(index), task.done = !task.done" clickable :class=" {'done': task.done }">
        <q-item-section avatar>
          <q-checkbox v-model="task.done" color="primary" class="no-pointer-events" />
        </q-item-section>
        <q-item-section>
          <q-item-label>{{ task.title }}</q-item-label>
        </q-item-section>
        <q-item-section v-if="task.done" side>
          <q-btn @click.stop="deleteTask(index)" round color="primary" icon="delete" dense></q-btn>
        </q-item-section>
      </q-item>
    </q-list>
  </q-page>
</template>

<script>
import Vue from 'vue'
import axios from 'axios'
import qs from 'querystring'

Vue.prototype.$axios = axios

const config = {
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded'
  }
}

export default {
  data () {
    return {
      newTask: '',
      tasks: []
    }
  },
  methods: {
    loadData () {
      this.tasks = []
      this.$axios.get('http://localhost:8081')
        .then((response) => {
          response.data.forEach(element => {
            const task = {
              title: element.Description,
              done: element.IsDone,
              id: element.ID
            }
            this.tasks.push(task)
          })
        })
        .catch(() => {
          this.$q.notify({
            color: 'negative',
            position: 'top',
            message: 'Loading failed',
            icon: 'report_problem'
          })
        })
    },
    addTask () {
      const requestBody = {
        description: this.newTask
      }
      const url = 'http://localhost:8081'

      this.$axios.post(url, qs.stringify(requestBody), config)
        .then((result) => {
          console.log(result)
        })
        .catch((err) => {
          console.log(err)
        })
      this.loadData()
      this.newTask = ''
    },
    setTask (index) {
      const id = this.tasks[index].id
      const requestBody = {
        isDone: !this.tasks[index].done
      }
      const url = 'http://localhost:8081/' + id

      this.$axios.patch(url, qs.stringify(requestBody), config)
        .then((result) => {
          console.log(result)
        })
        .catch((err) => {
          console.log(err)
        })
      this.loadData()
    },
    deleteTask (index) {
      const id = this.tasks[index].id
      const url = 'http://localhost:8081/' + id
      this.$axios.delete(url, config)
        .then((result) => {
          console.log(result)
        })
        .catch((err) => {
          console.log(err)
        })
      this.loadData()
    }
  },
  mounted () {
    this.loadData()
  }
}

</script>
<style lang="scss">
  .done {
    .q-item__label {
      text-decoration: line-through;
      color: #bbb;
    }
  }
</style>
