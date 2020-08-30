<template lang="pug">
  #app
    b-container(fluid='sm')
      h1.mb-5.mt-3 To Do List

      b-nav.mb-3
        <b-nav-item @click="$bvModal.show('modal-'); handleAdd();">Add New Todo</b-nav-item>

      b-table(outlined :busy="loading == true" :items="items" :fields="fields")

        <template v-slot:table-busy>
          <div class="text-center text-danger my-2">
            <b-spinner class="align-middle"></b-spinner>
            <strong>Loading...</strong>
          </div>
        </template>

        template(v-slot:cell(completed)="data")
          b-button(:variant="data.item.completed === true ? 'success' : 'warning'" @click="updateStatus(data.item)") {{ data.item.completed === true ? 'Uncomplete' : 'Complete' }}

        template(v-slot:cell(action)="data, ind")
          b-button-group 
            b-button(variant="info" @click="$bvModal.show('modal-'); handleEdit(data.item);") EDIT
            b-button(variant="danger" @click="handleDelete(data.item, data.index)") DELETE


      <b-modal :id='"modal-"' hide-footer>
        <template v-slot:modal-title>{{ form_type == 'update' ? 'Edit' : 'New' }}</template>
        <div class="d-block text-center">
          <b-row class="my-1">
            <b-col sm="2">
              <label for="form-title">Title:</label>
            </b-col>
            <b-col sm="10">
              <b-form-input id="form-title" placeholder="Title" v-model="form.title" required></b-form-input>
            </b-col>
          </b-row>
          <b-row class="my-1">
            <b-col sm="2">
              <label for="form-date">Date:</label>
            </b-col>
            <b-col sm="10">
              b-form-datepicker(id="form-date" v-model="form.date" required)
            </b-col>
          </b-row>
          <b-row class="my-1">
            <b-col sm="2">
              <label for="input-default">Complete?:</label>
            </b-col>
            <b-col sm="10">
              b-form-checkbox(
                id="checkbox-1"
                v-model="form.completed"
                name="checkbox-1")
            </b-col>
          </b-row>
        </div>
        <b-button class="mt-3" block @click="$bvModal.hide('modal-'); handleSend();" variant="success">Send</b-button>
      </b-modal>

</template>

<script>
import axios from 'axios'

export default {
  name: 'app',
  data () {
    return {
      fields: [{
        key: 'id',
        sortable: true,
      }, {
        key:'title',
        sortable: true,
      }, 'date', {
        key: 'completed',
        label: 'Status',
      },{
        key: 'action',
        label: 'Action',
      }],
      items: [],
      loading: false,
      form_type: 'update',
      form: {
        id: '',
        title: '',
        data: '',
        completed: '',
      }
    }
  },
  methods: {
    getList() {
      this.loading = true
      // Index
      axios.get('https://my-json-server.typicode.com/wsh-startup/mock-api/tasks').then((resp) => {
        this.loading = false
        this.items = resp.data
      })
    },
    handleSend() {
      this.loading = true

      if(this.form_type == 'update') {
        axios.put('https://my-json-server.typicode.com/wsh-startup/mock-api/tasks/' + this.form.id, this.form).then((resp) => {
          this.items[resp.data.id] = resp.data
          this.loading = false
        }).catch(e => {
          console.log(e)
          this.loading = false
        })
      }
      else if(this.form_type == 'create') {
        axios.post('https://my-json-server.typicode.com/wsh-startup/mock-api/tasks', this.form).then((resp) => {
          this.items.push(resp.data)
          this.loading = false
        }).catch(e => {
          console.log(e)
          this.loading = false
        })
      }
    },
    handleAdd() {
      this.form_type = 'create'

      this.form = {
        id: '',
        title: '',
        data: '',
        completed: '',
      }
    },
    handleEdit(data) {
      this.form_type = 'update'

      this.form = data
    },
    handleDelete(data, ind) {
      this.loading = true
      
      axios.delete('https://my-json-server.typicode.com/wsh-startup/mock-api/tasks/' + data.id).then((resp) => {
        const i = ind
        let _items = this.items.slice(0, i).concat(this.items.slice(i + 1, this.items.length))

        this.items = _items

        this.loading = false
      }).catch(e => {
        console.log(e)
        this.loading = false
      })
    },
    updateStatus(data) {
      data.completed = !data.completed
      this.form = data

      this.form_type = 'update'

      this.handleSend()
    }
  },
  created() {
    this.getList()
  }
}
</script>

<style lang="sass">
  table
    min-height: 100vh
</style>
