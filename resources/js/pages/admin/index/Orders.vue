<template>
    <div>
        <v-container>
            <div class="d-flex justify-space-between mb-6" >
                <div>
                    <h3> DELIVERY ORDERS </h3>
                </div>
                <div>
                    <v-menu
                        v-model="date_pick"
                        :close-on-content-click="false"
                        :nudge-right="40"
                        transition="scale-transition"
                        offset-y
                        min-width="auto"
                    >
                        <template v-slot:activator="{ on, attrs }">
                        <v-text-field
                            v-model="date_filter"
                            label="Select Date"
                            prepend-icon="mdi-calendar"
                            readonly
                             color="light-blue"
                            v-bind="attrs"
                            v-on="on"
                        ></v-text-field>
                        </template>
                        <v-date-picker
                            color="light-blue"
                            v-model="date_filter"
                            @input="date_pick = false"
                        ></v-date-picker>
                    </v-menu>  
                </div>
              
                <div>
                    <v-select
                        v-model="status_filter"
                        :items="status_filters"
                        menu-props="auto"
                        label="Status Filter"
                        color="light-blue"
                        hide-details
                        prepend-icon="mdi-filter-variant"
                        single-line
                    ></v-select>
                 
                </div>
                   <v-btn @click="generateDialog = true">
                        Generate Report
                    </v-btn>
            </div>
            <div class="d-flex justify-end" >
                <h3>
                    TOTAL SALES: ₱{{grand_total}}
                </h3> 
            </div>
            <v-row >
                <v-col cols='12'>
                    <div class="d-flex justify-end" >
                        <div class="pending mr-4 p-2 status-box text-white">
                            pending
                        </div>
                        <div class="accepted mr-4 p-2 status-box text-white">
                            accepted
                        </div>
                        <div class="denied mr-4 p-2 status-box text-white">
                            denied
                        </div>
                        <div class="assigned-to-driver mr-4 p-2 status-box text-white">
                            assigned-to-driver
                        </div>
                        <div class="on-the-way mr-4 p-2 status-box text-white">
                            on-the-way
                        </div>
                        <div class="delivered mr-4 p-2 status-box text-white">
                            delivered
                        </div>
                    </div>
                </v-col>

                <v-col cols="12" >
                    <v-expansion-panels focusable>
                        <v-expansion-panel
                        class="text-white"
                        v-for="(order,i) in orders"
                        :key="i"
                        :class="order.status"
                        v-if="orders.length > 0"
                        >
                            <v-expansion-panel-header >Order #{{order.id}} by {{order.user.name}}</v-expansion-panel-header>
                            <v-expansion-panel-content>
                                <v-row class="mt-1">
                                    <v-col cols="3">
                                        <p>Customer Details</p>
                                        <div class="d-flex ml-4" >
                                            <v-avatar>
                                                <img
                                                    :src="order.user.image ? '/storage/'+ order.user.image: 'https://upload.wikimedia.org/wikipedia/commons/7/71/Nothing_whitespace_blank.png'"
                                                    :alt="order.user.name"
                                                >
                                            </v-avatar>
                                            <div class="ml-4">
                                                <p class="m-0">{{order.user.name}}</p>
                                                <p class="m-0">{{order.user.email}}</p>
                                                <p class="m-0">{{order.user.phone_number}}</p>
                                                <p class="m-0">Purok: {{order.user.purok}}</p>
                                                <p class="m-0">Brgy: {{order.user.brgy}}</p>
                                                <p class="m-0">City: {{order.user.city}}</p>
                                                <p class="m-0">Landmark: {{order.user.landmark}}</p>
                                                <p class="m-0">Addtional Address: {{order.user.additional_address}}</p>
                                            </div>
                                        </div>
                                        <p> Delivery Man Details</p>
                                        <div class="d-flex" v-if="order.delivery_man_id">
                                            <v-avatar>
                                                <img
                                                    :src="order.delivery_man.image ? '/storage/'+ order.delivery_man.image: 'https://upload.wikimedia.org/wikipedia/commons/7/71/Nothing_whitespace_blank.png'"
                                                    :alt="order.delivery_man.name"
                                                >
                                            </v-avatar>
                                            <div class="ml-4">
                                                <p class="m-0">{{order.delivery_man.name}}</p>
                                                <p>{{order.delivery_man.username}}</p>
                                                <p>{{order.delivery_man.phone_number}}</p>
                                            </div>
                                        </div>
                                        <div v-else >
                                            <p  class="ml-2 text-grey">
                                                Not Assigned
                                            </p>
                                        </div>
                                        <p> Ordered Time: {{moment(order.created_at).calendar()}}</p>
                                        <p> Date to Deliver: {{order.date_to_deliver ? order.date_to_deliver : "Not Defined" }}</p>
                                        <p> Time to Deliver: {{order.time_to_deliver ? order.time_to_deliver : "Not Defined" }}</p>
                                        <div>
                                            Total Bill: ₱{{order.total}}
                                        </div>
                                        <v-btn
                                            block
                                            :class="order.status"
                                            @click="setOrderToBeUpdated(order)"
                                            class="text-white mt-2"
                                        >
                                            Update Status
                                        </v-btn>
                                    </v-col>
                                    <v-col cols="9">
                                        <v-simple-table height="440" >
                                            <template v-slot:default>
                                                <thead>
                                                    <tr>
                                                        <th class="text-left">
                                                            Product Name
                                                        </th>
                                                        <th class="text-left">
                                                            Description
                                                        </th>
                                                         <th class="text-left">
                                                            Product Price
                                                        </th>
                                                        <th class="text-left">
                                                            For Refill
                                                        </th>
                                                        <th class="text-left">
                                                            Quantity
                                                        </th>
                                                        <th class="text-left">
                                                            Total Price
                                                        </th>
                                                    </tr>
                                                </thead>
                                                <tbody>
                                                    <tr
                                                        v-for="item in order.orders"
                                                        :key="item.id"
                                                    >
                                                        <td>
                                                           <v-avatar>
                                                                <img
                                                                    :src="item.product.image ? '/storage/'+ item.product.image : 'https://upload.wikimedia.org/wikipedia/commons/7/71/Nothing_whitespace_blank.png'"
                                                                    :alt="item.product.name"
                                                                >
                                                            </v-avatar>
                                                            {{ item.product.name }}
                                                        </td>
                                                        <td>{{ item.product.description }}</td>
                                                        <td>₱{{ item.product.price }}</td>
                                                        <td>{{ item.product.is_refill ? "Yes" : "Container for sale" }}</td>
                                                        <td>{{ item.quantity }}</td>
                                                        <td>₱{{ item.total_price }}</td>
                                                    </tr>
                                                </tbody>
                                            </template>
                                        </v-simple-table>
                                    </v-col>
                                </v-row>
                            </v-expansion-panel-content>
                        </v-expansion-panel>
                    </v-expansion-panels>
                </v-col>
            </v-row>
            <v-row justify="center">
                <v-col cols="12">
                <v-container class="max-width">
                    <v-pagination
                    v-model="page"
                    color="light-blue"
                    class="my-4"
                    :length="pageNumbers"
                    ></v-pagination>
                </v-container>
                </v-col>
            </v-row>
        </v-container>
        <OrderForm :order="order_to_update" :dialogState="status_dailog" @close="close" />
        <v-dialog
            v-model="generateDialog"
            persistent
            max-width="600px"
            >
            
            <v-card>
                <v-card-title>
                    <span class="text-h5">Select Filter for Orders Report</span>
                </v-card-title>
                <v-card-text>
                    <v-container>
                        <v-row>
                            <v-col      
                                cols="7"
                                sm="7"
                                md="7"
                            >   
                               
                                <label for="">Filter start to end date</label>
                                <div>
                                    <v-date-picker
                                        color="light-blue"
                                        @input="generate_date_pick = false"
                                        v-model="generate_dates"
                                        range
                                    ></v-date-picker>
                                </div>
                            </v-col>
                            <v-col cols="5">
                                 <div>
                                    <label for="">Filter status</label>
                                    <v-select
                                        v-model="generate_status_filter"
                                        :items="status_filters"
                                        menu-props="auto"
                                        label="Status Filter"
                                        color="light-blue"
                                        hide-details
                                        prepend-icon="mdi-filter-variant"
                                        single-line
                                    ></v-select>
                                </div>
                            </v-col>
                        </v-row>
                    </v-container>
                </v-card-text>
                <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                        color="blue darken-1"
                        text
                        @click="generateDialog = false"
                    >
                        Close
                    </v-btn>
                    <v-btn
                        color="blue darken-1"
                        text
                        @click="downloadPDF"
                    >
                        Generate
                    </v-btn>
                </v-card-actions>
            </v-card>
        </v-dialog>
    </div>
</template>

<script>
import OrderForm from '../../../components/adminForms/Order.vue'
export default {
    data : () =>({
        orders: [],
        page : 1,
        grand_total: 0,
        pageNumbers: 1,
        date_pick: false,
        status_filters : ['on-the-way','assigned-to-driver', 'pending','delivered','accepted','denied','all'],
        status_filter : 'all',
        date_filter: (new Date()).toISOString().split('T')[0],
        status_dailog : false,
        order_to_update : {},

        generateDialog: false,
        generate_date_pick: false,
        generate_status_filter: 'all',
        generate_dates: [(new Date()).toISOString().split('T')[0],(new Date(new Date().getTime()+(14*24*60*60*1000))).toISOString().split('T')[0]],

    }),
    components : {
        OrderForm,
    },
    mounted () {
        this.initialize()
    },
    computed: {
    dateRangeText () {
        return this.generate_dates.join(' ~ ')
      },
    },
    methods : {
        initialize(){
            let params = { 
                page: this.page,
                per_page: 10,
                status_filter : this.status_filter,
                date_filter : this.date_filter
            } 

            this.$admin.get('/order/all', { params })
            .then(({data}) => {
                this.orders = data.orders.data;

                this.grand_total = data.grandtotal
                this.orders.forEach((order) => {
                    order.orders = JSON.parse(order.orders)
                })
                this.page = data.current_page;
                this.pageNumbers = data.last_page;
            });
        },
        setOrderToBeUpdated(order) {
            this.order_to_update = JSON.parse(JSON.stringify(order))
            this.status_dailog = true
        },
        close(){
            this.status_dailog = false
            this.initialize()
        },
        downloadPDF()
        {
            this.$admin.get('/order/generate-pdf/'+this.generate_dates[0]+'/'+this.generate_dates[1]+'/'+this.generate_status_filter,{
                responseType: 'arraybuffer'
            })
            .then(response => {
                const url = window.URL.createObjectURL(new Blob([response.data]));
                const link = document.createElement('a');
                link.href = url;
                link.setAttribute('download', 'DELORDERSREPORT'+this.generate_dates[0]+'-'+this.generate_dates[1]+'.pdf');
                document.body.appendChild(link);
                link.click();
            })
        }
    },
    watch : {
        page (){
            this.initialize()
        },
        status_filter() {
            this.initialize()
        },
        date_filter() {
            this.initialize()
        },
    }


    
}
</script>

<style lang="scss" scoped>
.on-the-way{
    background-color:rgb(233, 135, 151)!important;
}
.assigned-to-driver{
    background-color: rgb(216, 166, 72)!important;
}
.pending{
    background-color: rgb(192, 192, 56)!important;
}
.delivered{
    background-color: rgb(51, 115, 130)!important;
}
.accepted{
    background-color: rgb(64, 187, 64)!important;
}
.denied{
    background-color: rgb(194, 66, 66)!important;
}
.status-box{
    border-radius: 5%;
}
</style>
