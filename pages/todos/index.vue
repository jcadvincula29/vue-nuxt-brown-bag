<template>
  <div>
    <v-card
      max-width="475"
      class="mx-auto"
    >
      <v-toolbar
        color="teal"
        dark
      >
        <v-toolbar-title>Your List of Task</v-toolbar-title>
        <v-spacer />
        <v-btn
          color="teal"
          fab
          small
          dark
          class="ma-2"
          @click.stop="showForm()"
        >
          <v-icon>mdi-plus</v-icon>
        </v-btn>
      </v-toolbar>

      <v-list
        subheader
        two-line
        flat
      >
        <v-list-item-group>
          <v-list-item v-for="(todo, index) in todos" :key="index">
            <template>
              <v-list-item-action>
                <v-checkbox
                  v-model="todos[index].completed_at"
                  color="primary"
                  @click.stop="check(index)"
                />
              </v-list-item-action>

              <v-list-item-content>
                <v-list-item-title>
                  {{ todo.task }}
                  <v-icon
                    small
                    class="ml-3 mr-2"
                    @click="showForm(todo)"
                  >
                    mdi-pencil
                  </v-icon>
                  <v-icon
                    small
                    @click="deleteTask(todo.id)"
                  >
                    mdi-delete
                  </v-icon>
                </v-list-item-title>
                <v-list-item-subtitle>{{ todo.description }}</v-list-item-subtitle>
              </v-list-item-content>
            </template>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-card>

    <v-dialog v-model="dialog" max-width="500px">
      <v-card>
        <v-card-title>
          <span class="headline">{{ formTitle }}</span>
        </v-card-title>
        <v-card-text>
          <v-form>
            <v-container>
              <v-row>
                <v-col cols="12">
                  <v-text-field v-model="formParams.task" label="Task" />
                </v-col>

                <v-col cols="12">
                  <v-textarea v-model="formParams.description" label="Description" />
                </v-col>
              </v-row>
            </v-container>
          </v-form>
        </v-card-text>

        <v-card-actions>
          <v-spacer />
          <v-btn color="blue darken-1">
            Cancel
          </v-btn>
          <v-btn color="blue darken-1" @click.prevent="submit">
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  data () {
    return {
      finished: [],
      todos: [],
      dialog: false,
      id: null,
      formParams: {
        description: '',
        task: ''
      },
      forEdit: false
    }
  },
  computed: {
    formTitle () {
      return (this.forEdit ? 'Edit' : 'New') + ' Task'
    }
  },
  created () {
    this.listTask()
  },
  methods: {
    listTask () {
      this.$axios.get('/todos')
        .then((response) => {
          this.todos = response.data.data
        })
    },
    check (index) {
      const id = this.todos[index].id
      this.$axios.$put(`/todos/${id}/toggle`)
        .then((response) => {
          this.todos[index].completed_at = response.completed_at
        })
        .catch((error) => {
          // eslint-disable-next-line no-console
          console.log(error)
        })
    },
    showForm (item = null) {
      this.dialog = true
      if (item) {
        this.forEdit = true
        this.formParams.task = item.task
        this.formParams.description = item.description
        this.id = item.id
      }
    },
    submit () {
      let request

      if (this.forEdit) {
        request = this.$axios.$put(`/todos/${this.id}`, this.formParams)
      } else {
        request = this.$axios.$post('/todos', this.formParams)
      }

      request.then((response) => {
        this.dialog = false
        this.forEdit = false
        this.formParams.task = ''
        this.formParams.description = ''
        this.id = null
        this.$swal('Success', 'Submission Completed', 'success')
        this.listTask()
      })
        .catch(() => {
          this.$swal('Ooops!!', 'Error. Sorry for inconvenience', 'error')
        })
    },
    deleteTask (id) {
      this.$axios.$delete(`/todos/${id}`)
        .then((response) => {
          this.$swal('Success', 'Delete Successfully', 'success')
          this.listTask()
        })
        .catch(() => {
          this.$swal('Ooops!!', 'Error. Sorry for inconvenience', 'error')
        })
    }
  }
}
</script>
