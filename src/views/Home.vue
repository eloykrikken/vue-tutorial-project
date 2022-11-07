<template>
    <AddTask
        v-show="showAddTask" 
        @add-task="addTask"
    />
    <Tasks @toggle-reminder="toggleReminder" @delete-task="deleteTask" :tasks="tasks" />
</template>

<script>
import Tasks from '../components/Tasks'
import AddTask from '../components/AddTask'
export default {
    name: 'Home',
    props: {
        showAddTask: Boolean,
    },
    components: {
        Tasks,
        AddTask
    },
    data() {
        return {
            tasks: []
        }
    },
    methods: {
    async deleteTask(id) {
      if (confirm('Sure?')) {
        const res = await fetch(`api/tasks/${id}`, 
        {
          method: 'DELETE',
        })
        const data = await res.json();
        res.status === 200 ? 
          (this.tasks = this.tasks.filter(task => task.id !== id)) :
          alert('deletion failed');
      }
    },
    async toggleReminder(id) {
      const taskToToggle = await this.fetchTask(id);
      const updateTask = {...taskToToggle, reminder: !taskToToggle.reminder};
      const res = await fetch(`api/tasks/${id}`, {
        method: 'PUT',
        headers: {
          'Content-type': 'application/json'
        },
        body: JSON.stringify(updateTask)
      });

      const data = await res.json();

      for (const task of this.tasks) {
        if (task.id === id) {
          task.reminder = data.reminder
        }
      }
    },
    async addTask(task) {
      const res = await fetch('api/tasks', 
        {
          method: 'POST',
          headers: {
            'Content-type': 'application/json'
          } ,
          body: JSON.stringify(task)       
        }
      )
      const data = await res.json()
      this.tasks.push(data);
    },
    async fetchTasks() {
      const res = await fetch('api/tasks');

      const data = await res.json()

      return data
    },

    async fetchTask(id) {
      const res = await fetch(`api/tasks/${id}`);

      return await res.json();
    } 
  },
  async created() {
    this.tasks = await this.fetchTasks()
  }
}
</script>
