<template>
  <v-card>
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
                    <v-text-field class="shrink" v-model="search" append-icon="mdi-magnify" label="Search" outlined dense hide-details></v-text-field>

                    <v-dialog v-model="dialog" max-width="800">
                        <template v-slot:activator="{on ,attrs}">
                            <v-btn class="ml-2" color="primary" v-bind="attrs" v-on="on" dark>
                                New
                            </v-btn>
                        </template>
                        
                        <v-card>
                            <v-card-title>{{ formTitle }}</v-card-title>
                            <v-divider></v-divider>
                            <v-card-text class="pa-2">
                                <v-form>
                                    <v-container>
                                      <v-row>
                                        <v-col cols="6">
                                          <v-row>
                                            <v-col>
                                              <v-text-field outlined dense required v-model="patient.firstname" label="First name"></v-text-field>
                                              <v-text-field outlined dense required v-model="patient.middlename" label="Middle name"></v-text-field>
                                              <v-text-field outlined dense required v-model="patient.lastname" label="Last name"></v-text-field>
                                            </v-col>
                                          </v-row>
                                        </v-col>
                                        <v-divider inset vertical></v-divider>
                                        <v-col cols="6">
                                          <v-row no-gutters>
                                            <v-col cols="4" class="pr-2">
                                              <v-text-field outlined dense required v-model="patient.age" label="Age"></v-text-field>
                                            </v-col>
                                            <v-col cols="8">
                                              <v-radio-group row class="ma-1">
                                                <v-radio value="yrs" label="Yrs."></v-radio>
                                                <v-radio value="mos" label="Mos."></v-radio>
                                              </v-radio-group>
                                            </v-col>
                                          </v-row>

                                          <v-row no-gutters>
                                            <v-col cols="6" class="pr-2">
                                              <v-select outlined dense v-model="patient.gender" :items="gender" item-text="text" item-value="value" label="Gender"></v-select>
                                            </v-col>
                                            <v-col cols="6">
                                              <v-text-field outlined dense required v-model="patient.contact" label="Contact #."></v-text-field>
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
                                <v-btn text color="primary" @click="btn_save">Save</v-btn>
                                <v-btn text color="error" @click="close">Cancel</v-btn>
                            </v-card-actions>
                        </v-card>
                    </v-dialog>

                </v-toolbar>
            </template>

            <template v-slot:[`item.status`]="{ item }">
                <v-chip v-if="item.status == 'P'" color="green" small dark link @click="getItemId(item)">
                  ONGOING
                </v-chip>
                <v-chip v-else-if="item.status == 'F'"  color="warning" small dark link @click="getItemId(item)">
                  FOR PRINT
                </v-chip>
                <v-chip v-else-if="item.status == 'C'"  color="info" small dark link @click="getItemId(item)">
                  CREATE
                </v-chip>
            </template>

            <template v-slot:[`item.actions`]="{ item }">
                <v-btn class="mr-2" dense x-small @click="btn_update(item)" color="primary">UPDATE</v-btn>
                <v-btn dense x-small @click="btn_delete(item)" color="error">DELETE</v-btn>
            </template>

            <template v-slot:[`item.gender`]="{ item }">
                {{ item.gender == "m" ? "Male" : "Female" }}
            </template>

            <template v-slot:[`item.firstname`]="{ item }">
                <p class="text-uppercase ma-0">{{ item.firstname }}</p>
            </template>
            <template v-slot:[`item.lastname`]="{ item }">
                <p class="text-uppercase ma-0">{{ item.lastname }}</p>
            </template>
            <template v-slot:[`item.middlename`]="{ item }">
                <p class="text-uppercase ma-0">{{ item.middlename }}</p>
            </template>

        </v-data-table>
    </v-flex>


  </v-card>

  
</template>

<script>

export default {
  name: 'Appointments',

  data(){
        return{
            overlay: false,
            snackbar: false,
            snackbartext: '',
            snackbartimeout: -1,
            snackbarcolor: '',
            search: '',
            dialog: false,
            itemIndex: -1,
            table_header: [
                { text: 'Patient status', align: 'center', value: 'status', filterable: false },
                { text: 'Last Name', value: 'lastname' },
                { text: 'First Name', value: 'firstname' },
                { text: 'Middle Name', value: 'middlename' },
                { text: 'Age', align: 'center', value: 'age', filterable: false  },
                { text: 'Gender', align: 'center', value: 'gender', filterable: false  },
                { text: 'Created date', align: 'center', value: 'created_at', filterable: false  },
                // { text: 'Last Checkup', align: 'center', value: 'last_checkup', filterable: false  },
                { text: 'Actions', align: 'center', value: 'actions', filterable: false  },
            ],
            table_items: [],
            gender: [
              { text: 'Male', value: 'm' },
              { text: 'Female', value: 'f' }
            ],
            agetype: [
              { text: 'yrs' , value: 'yrs' },
              { text: 'mos', value: 'mos'}
            ],
            patient:{
              id: null, firstname: null, lastname: null, middlename: null, age: null, gender: null, contact: null, address: null
            },
            defaultpatient:{
              id: null, firstname: null, lastname: null, middlename: null, age: null, gender: null, contact: null, address: null
            }
        }
    },
  beforeCreate: function(){
      if(!this.$session.has('user-session')){
          this.$router.push('/login');
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
    // btn_event: function(event_name, item){
    //   console.log(event_name)
    //   console.log(item.id)
    // }

    async loaditems(){
      this.overlay = true
      await this.$guest.get('/api/patient/getPatients')
      .then(res => {
        this.table_items = res.data
        this.overlay = false
      })
      .catch(err =>{ console.log(err) })
    },  

    async btn_save(){
      let data = {
        id: this.patient.id,
        firstname: this.patient.firstname,
        lastname: this.patient.lastname,
        middlename: this.patient.middlename,
        age: this.patient.age,
        gender: this.patient.gender,
        contact: this.patient.contact,
        address: this.patient.address
      }

      if(this.itemIndex == -1){
        await this.$guest.post('/api/patient/insertPatient', this.$form_data.generate(data))
        .then(res => {
          console.log(res.data)
          this.loaditems()
          this.close()
        })
        .catch(err => { console.log(err) })
      }else{
        await this.$guest.post('/api/patient/updatePatient', this.$form_data.generate(data))
        .then(res => {
          console.log(res.data)
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

    getItemId(item){
      // this.$router.push({ name: 'Patient_form', params: { patient_info: item } })
      this.$router.push({ name: 'Patient_form', query: { id: item.id, stat: item.status, cdate: item.last_checkup } })
    },

    close(){
      this.dialog = false;
      this.$nextTick(() => {
        this.patient = Object.assign({}, this.defaultpatient)
        this.itemIndex = -1
      })
    }
  }
}
</script>

<style>

</style>