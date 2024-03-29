<template>
<div class="ma-n3">
    <myHeader :title="'Patient Lists'" :subtitle="'Create Patient Records'" />
    <v-container fluid>
  <v-card flat outlined>
    <!-- <customTable :table_search="search" :table_header="table_header" :table_items="table_items" :btn_items="btn_items" @btn_event="btn_event"/> -->
    <v-flex md12 class="ma-2">
      <v-overlay :value="overlay">
        <v-progress-circular indeterminate size="70"></v-progress-circular>
      </v-overlay>

      <v-data-table dense flat disable-sort 
        :items-per-page="15"
        :headers="table_header"
        :items="table_items"
        :search="search">
            <template v-slot:top>
                <v-toolbar flat>
                    <v-spacer></v-spacer>
                    <v-text-field class="shrink" v-model="search" clearable append-icon="mdi-magnify" label="Search" outlined dense hide-details></v-text-field>

                    <v-dialog v-model="dialog" max-width="800">
                        <template v-slot:activator="{on ,attrs}">
                            <v-btn class="ml-2" color="primary" v-bind="attrs" v-on="on" dark>
                                Add Patient
                            </v-btn>
                        </template>
                        
                        <v-card>
                            <v-card-title>{{ formTitle }}</v-card-title>
                            <v-divider></v-divider>
                            <v-card-text class="pa-2">
                                <v-form ref="form">
                                    <v-container>
                                      <v-row>
                                        <v-col cols="6">
                                          <v-row>
                                            <v-col>
                                              <v-text-field outlined dense required v-model="patient.lastname" label="Last name" :rules="lastnameRules"></v-text-field>
                                              <v-text-field outlined dense required v-model="patient.firstname" label="First name" :rules="firstnameRules"></v-text-field>
                                              <v-text-field outlined dense required v-model="patient.middlename" label="Middle name"></v-text-field>
                                            </v-col>
                                          </v-row>
                                        </v-col>
                                        <v-divider inset vertical></v-divider>
                                        <v-col cols="6">
                                          <v-row no-gutters>
                                            <v-col cols="4" class="pr-2">
                                              <v-text-field outlined dense required v-model="patient.age" v-mask="'##'" label="Age" :rules="ageRules"></v-text-field>
                                            </v-col>
                                            <v-col cols="8">
                                              <v-radio-group v-model="patient.agetype" row class="ma-1">
                                                <v-radio v-for="(item,i) in radioGroup" :key="i" :label="item.text" :value="item.value"></v-radio>
                                                <!-- <v-radio value=0 label="Mos."></v-radio> -->
                                              </v-radio-group>
                                            </v-col>
                                          </v-row>

                                          <v-row no-gutters>
                                            <v-col cols="6" class="pr-2">
                                              <v-select outlined dense v-model="patient.gender" :items="gender" item-text="desc" item-value="id" label="Gender" required :rules="genderRules"></v-select>
                                            </v-col>
                                            <v-col cols="6">
                                              <v-text-field outlined dense required v-mask="'####-###-####'" v-model="patient.contact" label="Contact #."></v-text-field>
                                            </v-col>
                                          </v-row>
                                          <v-text-field outlined dense required v-model="patient.address" label="Address"></v-text-field>
                                        </v-col>
                                      </v-row>
                          
                                    </v-container>
                                </v-form>
                            </v-card-text>
                            <v-divider></v-divider>
                            <v-card-actions>
                                <v-spacer></v-spacer>
                                <v-btn outlined color="primary" @click="btn_save">Save</v-btn>
                                <v-btn outlined color="error" @click="close">Cancel</v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>
                </v-toolbar>
            </template>
            <template v-slot:[`item.fullname`]="{ item }">
              {{ item.lastname + ", " + item.firstname + " " + item.middlename }}
            </template>
            <template v-slot:[`item.age`]="{ item }">
              {{ formatAge(item.age, item.agetype) }}
            </template>
            <template v-slot:[`item.gender`]="{ item }">
              {{ gender.filter(e => e.id == item.gender)[0]['desc'] }}
            </template>
            <template v-slot:[`item.last_checkup`]="{ item }">
              {{ formatDateTime(item.last_checkup) }}
            </template>
            <template v-slot:[`item.actions`]="{ item }">
                <v-btn class="mr-2" x-small @click="btn_update(item)" color="primary">UPDATE</v-btn>
                <v-btn x-small @click="btn_delete(item)" color="error">DELETE</v-btn>
            </template>
        </v-data-table>
    </v-flex>

  <Snackbar :snackbar="snackbar" />
  </v-card>
  </v-container>
</div>
</template>

<script>
import myHeader from '../components/myHeader.vue'
import Snackbar from '../components/Snackbar.vue'
export default {
  name: 'Appointments',
  components: { myHeader,Snackbar },
  data(){
        return{
            overlay: false,
            snackbar: {
              status: false,
              text: '',
              color: '',
            },
            search: '',
            dialog: false,
            itemIndex: -1,
            firstnameRules: [v => !!v || "First name is required!"],
            lastnameRules: [v => !!v || "Last name is required!"],
            ageRules: [v => !!v || "Age is required!"],
            genderRules: [v => !!v || "Gender is required!"],
            table_header: [
                { text: 'ID #', value: 'id',align: 'center' },
                { text: 'Last Name', value: 'lastname' },
                { text: 'First Name', value: 'firstname' },
                { text: 'Middle Name', value: 'middlename' },
                { text: 'Age', align: 'center', value: 'age', filterable: false  },
                { text: 'Gender', align: 'center', value: 'gender', filterable: false  },
                { text: 'Contact #', align: 'center', value: 'contact', filterable: false  },
                { text: 'Actions', align: 'center', value: 'actions', filterable: false  },
            ],
            table_items: [],
            gender: [],
            radioGroup: [
              { text: 'Years' , value: "yrs" },
              { text: 'Mos.', value: "mos" }
            ],
            patient:{
              id: '', firstname: '', lastname: '', middlename: '', age: '', agetype: 'yrs', gender: '', contact: '', address: ''
            },
            defaultpatient:{
              id: '', firstname: '', lastname: '', middlename: '', age: '', agetype: 'yrs', gender: '', contact: '', address: ''
            }
        }
    },
  beforeCreate: function(){
      if(!this.$session.has('user-session')){
          this.$router.push({ path: '/login' });
      }
      let user_access = this.$session.get('user-access')
        let cpath = this.$route.path
        let modpath = []
        user_access.forEach(el => {
          modpath.push(el.mod_path)
        })
        if(modpath.indexOf(cpath) == -1){
          this.$router.push({ path: '/login' })
        }
  },

  created(){
    this.loaditems()
  },

  computed:{
    formTitle(){
      return this.itemIndex === -1 ? 'New Patient' : 'Update Patient'
    }
  },

  methods: {
    async loaditems(){
      this.overlay = true
      await this.$guest.get('/api/data_maintenance/getGender')
      .then(res => {
        this.gender = res.data
      })
      .catch(err =>{ console.log(err) })
      
      await this.$guest.get('/api/patient/getPatients')
      .then(res => {
        this.table_items = res.data
        this.overlay = false
      })
      .catch(err =>{ console.log(err) })
    },  

    async btn_save(){
      if(this.$refs.form.validate()){
        let data = {
        id: this.patient.id,
        firstname: this.patient.firstname,
        lastname: this.patient.lastname,
        middlename: this.patient.middlename,
        age: this.patient.age,
        agetype: this.patient.agetype,
        gender: this.patient.gender,
        contact: this.patient.contact,
        address: this.patient.address,
        user_id: this.$session.get('userid-session')
      }

      let url = this.itemIndex == -1 ? '/api/patient/insertPatient' : '/api/patient/updatePatient'
      await this.$guest.post(url, this.$form_data.generate(data))
        .then(() => {
          this.snackbar.status = true
          this.snackbar.color = "success"
          this.snackbar.text = this.itemIndex == -1 ? 'Record(s) saved successfully!' : 'Record(s) updated successfully!'
          this.loaditems()
          this.close()
        })
        .catch(err => { console.log(err) })
      }
    },
    
    btn_update(item){
      this.itemIndex = this.table_items.indexOf(item)
      this.patient = Object.assign({}, item)
      this.dialog = true
    },
    
    btn_delete(item){
      console.log(item)
      this.snackbar = true
    },
    close(){
      this.dialog = false;
      this.$refs.form.reset()
      this.$nextTick(() => {
        this.patient = Object.assign({}, this.defaultpatient)
        this.itemIndex = -1
      })
    },
  }
}
</script>

<style>

</style>