<template>
  <b-row>
    <b-col></b-col>
    <b-col cols="12">

        <b-modal no-close-on-esc	 no-close-on-backdrop ref="confirmModalRef">
          <div slot="modal-header">
            <span class="modal_title">Confirmation</span>
          </div>
          {{ confirmModal.msg }}
          <div slot="modal-footer">
            <b-button @click.stop.prevent="confirmYes()" variant="success" class="mr-1">Yes</b-button>
            <b-button @click.stop.prevent="confirmNo()" variant="danger" class="mr-1">No</b-button>
          </div>
        </b-modal>
        <b-row>
          <b-col cols="6">
            <nuxt-link to="/todo/create">
              <b-button size="sm" class="mr-1" variant="success">
                New
              </b-button>
            </nuxt-link>
          </b-col>
          <b-col>
            <b-form-group horizontal class="mb-1">
              <b-input-group>
                <b-form-input v-model="filter" placeholder="Type to Search" />
                <b-input-group-append>
                  <b-btn :disabled="!filter" @click="filter = ''">Clear</b-btn>
                </b-input-group-append>
              </b-input-group>
            </b-form-group>
          </b-col>
        </b-row>

       <b-table @filtered="onFiltered" :filter="filter" :current-page=currentPage :per-page="perPage" striped hover responsive :items="todos" :fields="fields">
         <template slot="actions" slot-scope="row">
          <nuxt-link :to="{ path: '/todo/update/'+row.item._id }">
            <b-button size="sm" class="mr-1" variant="primary">
              Update
            </b-button>
          </nuxt-link>
          <b-button size="sm" @click.stop="deleteItem(row.index, row.item)" class="mr-1" variant="danger">
            Delete
          </b-button>
         </template>
       </b-table>
       <b-row>
          <b-col md="6" class="my-1">
            <b-pagination :total-rows="totalRows" :per-page="perPage" v-model="currentPage" class="my-0" />
          </b-col>
        </b-row>
       <b-alert :show="alert.dismissCountDown"
             dismissible
             variant="danger"
             @dismissed="alert.dismissCountDown=0"
             @dismiss-count-down="countDownChanged">
          {{ alert.msg }} {{ alert.dismissCountDown }} seconds...
        </b-alert>
    </b-col>
    <b-col></b-col>
  </b-row>
</template>

<script>
var querystring = require('querystring');
var Cookie = process.client ? require('js-cookie') : undefined

export default {
  middleware: 'authenticated',
  asyncData () {
    return {
      filter: '',
      currentPage: 1,
      perPage: 5,
      totalRows: 0,
      pageOptions: [ 5, 10, 15 ],
      alert:{
        dismissSecs: 5,
        dismissCountDown: 0,
        msg: ''
      },
      fields: {
        name:{
          label: 'name',
          sortable: true
        },
        author:{
          label: 'author',
          sortable: true,
          key: 'author.name'
        },
        done:{
          label: 'done',
          sortable: true
        },
        actions:{
          label: 'action',
          sortable: false
        }
      },
      todos:[{
          name: '',
          done: '',
          _id: '',
          description: '',
          author: [{}]
      }],
      confirmModal:{
        msg: '',
        _id: false,
        show: false
      }
    }
  },
  watch:{
    'confirmModal.show': function (newVal, oldVal) {
      if (newVal == true && oldVal == false) {
        this.$refs.confirmModalRef.show();
      }else if(newVal == true && oldVal == false){
        this.$refs.confirmModalRef.hide();
      }else{
        this.show = false;
        this.$refs.confirmModalRef.hide();
      }
    }
  },
  methods: {
    countDownChanged (dismissCountDown) {
      this.alert.dismissCountDown = dismissCountDown
    },
    showAlert () {
      this.alert.dismissCountDown = this.alert.dismissSecs
    },
    onFiltered (filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length
      this.currentPage = 1
    },
    fetchTodos(){
      var i = this;
      i.$axios.setHeader('Authentication', this.$store.state.auth.token);
      this.$axios.get('/todo').then(function (response) {
        if (response.data.error == true) {
          alert(response.data.msg)
        }else{
          i.totalRows = response.data.todos.length;
          i.todos = response.data.todos;
        }
      });;
    },
    deleteItem (index, item) {
      this.confirmModal._id = item._id;
      this.confirmModal.msg = 'Do You want to delete the todo '+ item.name + '?';
      this.confirmModal.show = true;
    },
    resetConfirmModal(){
      this.confirmModal.msg = '';
      this.confirmModal.show = false;
      this.confirmModal._id = false;
    },
    confirmYes(){
      var i = this;
      i.$axios.setHeader('Authentication', this.$store.state.auth.token);
      this.$axios.delete('/todo/delete/'+i.confirmModal._id).then(function (response) {
        if (response.data.error == false) {
          i.fetchTodos();
          i.resetConfirmModal();
          i.alert.msg = response.data.msg;
          i.showAlert();
        }
      })
    },
    confirmNo(){
      this.resetConfirmModal();
    }
  },
  mounted(){
    this.fetchTodos();
  }
}
</script>

<style>
.modal_title{
  font-size: 22px;
  font-weight: 600;
}
</style>
