<template>
    <div class="ma-n3">
        <myHeader :title="'DTR In/Out'" :subtitle="'Report Module Generation'" />
        <v-container fluid>
            <Overlay :value="overlay.value" />
            <v-row no-gutters class="d-flex justify-end align-end mb-4">
                <v-col v-if="reports.employees.length > 0">
                    <v-btn small color="primary" @click="print()">PRINT</v-btn>
                    <v-btn small color="primary" @click="test()">TEST</v-btn>
                    <download-excel :data="excel_items" :fields="excel_fields" name="dtrinout" :before-generate="startDL" :before-finish="endDL"><a href="#" @click.prevent="exportToExcel()">Export to Excel</a></download-excel>
                </v-col>
                <v-col cols="2" class="mr-2">
                    <v-select v-model="filters.year" :items="options.year" item-text="desc" item-value="id" @change="getPayperiod()" dense outlined hide-details></v-select>
                </v-col>
                <v-col cols="2">
                    <v-select v-model="filters.payperiod_id" :items="options.payperiod" item-text="pperiod" item-value="id" @change="getDtrinout()" dense outlined hide-details></v-select>
                </v-col>
            </v-row>
            <v-alert v-if="alert_status == false" type="error" outlined text dense>
                No Records Found!!!
            </v-alert>
            <v-row no-gutters>
                <v-col id="print-form">
                    <v-card v-for="(item,i) in reports.employees" :key="i" outlined class="mb-2">
                        <!-- <v-card-subtitle class="mb-n4">
                            <ul style="list-style: none; padding: 0;">
                                <li>
                                    <ul>
                                        <li style="width: 100px;">Full Name:</li>
                                        <li><strong>{{ fullname(item) }}</strong></li>
                                    </ul>
                                    <ul>
                                        <li style="width: 100px;">Position:</li>
                                        <li>{{ item.position_desc }}</li>
                                    </ul>
                                    <ul>
                                        <li style="width: 100px;">Payperiod:</li>
                                        <li style="color: red;">{{ reports.payperiod['pperiod'] + " => " + reports.payperiod['cfrom'].substr(5,5) + " - " + reports.payperiod['cto'].substr(5,5) }}</li>
                                    </ul>
                                </li>
                            </ul>
                        </v-card-subtitle> -->
                        <v-card-text>
                            <v-data-table :headers="table_headers" :items="table_items(item.id)" :items-per-page="-1" hide-default-header hide-default-footer dense>
                                <template v-slot:header="{}">
                                    <tr>
                                        <th colspan="3" style="text-align: left; padding-left: 2px">Full Name:</th>
                                        <th colspan="12" style="text-align: left;">{{ fullname(item) }}</th>
                                    </tr>    
                                    <tr>
                                        <th colspan="3" style="text-align: left; padding-left: 2px">Position:</th>
                                        <th colspan="12" style="text-align: left;">{{ item.position_desc }}</th>
                                    </tr> 
                                    <tr>
                                        <th colspan="3" style="text-align: left; padding-left: 2px">Payperiod:</th>
                                        <th colspan="12" style="text-align: left; color: red;">{{ reports.payperiod['pperiod'] + " => " + reports.payperiod['cfrom'].substr(5,5) + " - " + reports.payperiod['cto'].substr(5,5) }}</th>
                                    </tr> 
                                    <tr style="border-bottom:1px solid black">
                                        <th colspan="100%"></th>
                                    </tr>
                                    <tr>
                                        <th class="bgcolor" colspan="3"></th>
                                        <th class="bgcolor" colspan="4">Original Schedule</th>
                                        <th class="bgcolor" colspan="4">Encoded Time</th>
                                        <th class="bgcolor" colspan="2">Over Time</th>
                                        <th class="bgcolor" colspan="2">Under Time</th>
                                    </tr>
                                    <tr>
                                        <th class="bgcolor" style="width: 110px;">Date</th>
                                        <th class="bgcolor" style="width: 40px;">Day</th>
                                        <th class="bgcolor" style="width: 30px;">Type</th>
                                        <th class="bgcolor" style="width: 70px;">AM In</th>
                                        <th class="bgcolor" style="width: 70px;">AM Out</th>
                                        <th class="bgcolor" style="width: 70px;">PM In</th>
                                        <th class="bgcolor" style="width: 70px;">PM Out</th>
                                        <th class="bgcolor" style="width: 70px;">AM In</th>
                                        <th class="bgcolor" style="width: 70px;">AM Out</th>
                                        <th class="bgcolor" style="width: 70px;">PM In</th>
                                        <th class="bgcolor" style="width: 70px;">PM Out</th>
                                        <th class="bgcolor" style="width: 70px;">Start</th>
                                        <th class="bgcolor" style="width: 70px;">End</th>
                                        <th class="bgcolor" style="width: 70px;">Start</th>
                                        <th class="bgcolor" style="width: 70px;">End</th>
                                    </tr>
                                </template>
                                <template v-slot:body={}>
                                    <tr v-for="td in table_items(item.id)" :key="td.id">
                                        <td>{{ td.date }}</td>
                                        <td>{{ td.day }}</td>
                                        <td>{{ td.type }}</td>
                                        <td>{{ td.cws_amin.substr(11,5) == '00:00' ? td.sched_amin.substr(11,5) : td.cws_amin.substr(11,5) }}</td>
                                        <td>{{ td.cws_amout.substr(11,5) == '00:00' ? td.sched_amout.substr(11,5) : td.cws_amout.substr(11,5) }}</td>
                                        <td>{{ td.cws_pmin.substr(11,5) == '00:00' ? td.sched_pmin.substr(11,5) : td.cws_pmin.substr(11,5) }}</td>
                                        <td>{{ td.cws_pmout.substr(11,5) == '00:00' ? td.sched_pmout.substr(11,5) : td.cws_pmout.substr(11,5) }}</td>
                                        <td>{{ td.encoded_amin == '0000-00-00 00:00:00' ? '' : td.encoded_amin.substr(11,5) }}</td>
                                        <td>{{ td.encoded_amout == '0000-00-00 00:00:00' ? '' : td.encoded_amout.substr(11,5) }}</td>
                                        <td>{{ td.encoded_pmin == '0000-00-00 00:00:00' ? '' : td.encoded_pmin.substr(11,5) }}</td>
                                        <td>{{ td.encoded_pmout == '0000-00-00 00:00:00' ? '' : td.encoded_pmout.substr(11,5) }}</td>
                                        <td>{{ td.ot_start == '0000-00-00 00:00:00' ? '' : td.ot_start.substr(11,5) }}</td>
                                        <td>{{ td.ot_end == '0000-00-00 00:00:00' ? '' : td.ot_end.substr(11,5) }}</td>
                                        <td>{{ td.ut_start == '0000-00-00 00:00:00' ? '' : td.ut_start.substr(11,5) }}</td>
                                        <td>{{ td.ut_end == '0000-00-00 00:00:00' ? '' : td.ut_end.substr(11,5) }}</td>
                                    </tr>
                                </template>
                            </v-data-table>
                        </v-card-text>
                    </v-card>
                </v-col>
            </v-row>
        </v-container>
    </div>
</template>

<script>
import myHeader from '../../../components/myHeader.vue'
import Overlay from '../../../components/Overlay.vue'
export default {
    components: { myHeader, Overlay },
    data(){
        return{
            alert_status: true,
            overlay: {
                value: false
            },
            filters:{   
                year: null,
                payperiod_id: null
            },
            options:{
                year:[],
                payperiod:[
                    { id: null, pperiod: 'Please Select Payperiod...'},
                ]
            },
            reports:{
                employees: [],
                dtrinout: [],
                payperiod: []
            },
            table_headers:[
                { value: 'date', align: 'center' },
                { value: 'day', align: 'center' },
                { value: 'type', align: 'center' },
                { value: 'sched_amin', align: 'center' },
                { value: 'sched_amout', align: 'center' },
                { value: 'sched_pmin', align: 'center' },
                { value: 'sched_pmout', align: 'center' },
                { value: 'encoded_amin', align: 'center' },
                { value: 'encoded_amout', align: 'center' },
                { value: 'encoded_pmin', align: 'center' },
                { value: 'encoded_pmout', align: 'center' },
                { value: 'ot_start', align: 'center' },
                { value: 'ot_end', align: 'center' },
                { value: 'ut_start', align: 'center' },
                { value: 'ut_end', align: 'center' },
            ],
            excel_items: [
                { col1: 'item 1', col2: 'item 1', col3: 'item 1' },
                { col1: 'item 2', col2: 'item 2', col3: 'item 2' },
                { col1: 'item 3', col2: 'item 3', col3: 'item 3' },
            ],
            excel_fields: {
                fullname: 'col1',
                position: 'col2',
                payperiod: 'col3',
            }
        }
    },
    created(){
        this.initialize()
    },
    methods:{
        initialize(){
            this.overlay.value = true
            this.$guest('/api/reports/getYear')
            .then(res => {
                this.overlay.value = false
                this.options.year = res.data
                this.options.year.splice(0,0, Object.assign({}, { id: null, desc: 'Please Select Year...'}))
            })
            .catch(err => { console.log(err) })
        },
        getPayperiod(){
            if(this.filters.year != null){
                let data = {
                    year: this.filters.year,
                    payperiod_id: this.filters.payperiod_id
                }
                this.$guest.post('/api/reports/getPayperiod', this.$form_data.generate(data))
                .then(res => {
                    this.options.payperiod = res.data
                    this.options.payperiod.splice(0,0, Object.assign({}, { id: null, pperiod: 'Please Select Payperiod...'}))
                })
                .catch(err => { console.log(err) })
            }else{
                this.filters.payperiod_id = null
                this.options.payperiod = [{ id: null, pperiod: 'Please Select Payperiod...'}]
                this.alert_status = true
            }
        },

        async getDtrinout(){
            this.overlay.value = true
            let data = {
                year: this.filters.year,
                payperiod_id: this.filters.payperiod_id
            }
            await this.$guest.post('/api/reports/getDtrinout', this.$form_data.generate(data))
            .then(res => {
                this.overlay.value = false
                this.alert_status = res.data.status
                if(res.data.status == true){
                    this.reports.employees = res.data.employees
                    this.reports.payperiod = res.data.payperiod
                    this.reports.dtrinout = res.data.dtrinout
                }else{
                    this.reports.employees = []
                    this.reports.payperiod = []
                    this.reports.dtrinout = []
                }
            })  
            .catch(err => { console.log(err) })
        },
        fullname(item){
            let firstname = item.firstname.charAt(0).toUpperCase() + item.firstname.slice(1)
            let lastname = item.lastname.charAt(0).toUpperCase() + item.lastname.slice(1)
            let middlename = item.middlename.charAt(0).toUpperCase() + item.middlename.slice(1)
            return lastname + ", " + firstname + " " + middlename
        },
        table_items(item){
            return this.reports.dtrinout.filter(e => e.employee_id == item)
        },
        print(){
            this.print_form()
        },
        exportToExcel(){
        },
        test(){
            console.log()
        },
        startDL(){
            this.overlay.value = true;
        },
        endDL(){
            this.overlay.value = false;
        }
    }
}
</script>

<style scoped>
span{
    color: rgb(25, 118, 210);
}
.v-data-table{
    line-height: 1;
}
ul li{
    font-size: 13px;
    display: inline-block;
    list-style: none;
    line-height: 1rem;
}
tr td{
    text-align: center;
}
.bgcolor{
    background-color: rgb(217, 248, 192);
}
</style>