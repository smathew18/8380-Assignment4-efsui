<template>
  <v-main>
    <v-container fluid>
    <v-row align="center" justify="center">
        <v-col cols="12" align="center" justify="center">
          <blockquote>
            Welcome {{validUserName}}!
            <footer>
              <small>
                <em>&mdash;Eagle Financial Services, your Midwest Financial Services Partner.</em>
              </small>
            </footer>
          </blockquote>
        </v-col>
      <v-col  cols="12" md="10" lg="10" align="center" justify="center">
       <v-alert v-if="showMsg === 'new'"
                dismissible
                :value="true"
                type="success"
      >
        New fund has been added.
      </v-alert>
      <v-alert v-if="showMsg === 'update'" dismissible
        :value="true"
        type="success"
      >
        Fund information has been updated.
      </v-alert>
      <v-alert v-if="showMsg === 'deleted'" dismissible
              :value="true"
              type="success"
      >
        Selected Fund has been deleted.
      </v-alert>
    </v-col>
    </v-row>

      <!-- Data table -->
   <v-row align="center" justify="center">
    <v-col cols="12" md="10" v-resize="onResize">
      <v-data-table
            :headers="headers"
            :items="funds"
            class="elevation-1"
            fixed
            style="max-height: 300px; overflow-y: auto"
            v-if="!isMobile"
        >
         <template v-slot:item="props">
           <tr>
                        <td align="left">{{ props.item.cust_number }}</td>
                        <td nowrap="true" align="left">{{ props.item.category }}</td>
                        <td nowrap="true" align="left">{{ props.item.fund_name }}</td>
                        <td nowrap="true" align="left">{{ props.item.fund_acquired_value }}</td>
                        <td nowrap="true" align="left">{{ props.item.fund_acquired_date }}</td>
                        <td nowrap="true" align="left">{{ props.item.fund_recent_value }}</td>
                        <td nowrap="true" align="left">{{ props.item.fund_recent_date }}</td>
                        <td align="center"><v-icon @click="updateFund(props.item)">mdi-pencil</v-icon></td>
                        <td align="center"><v-icon @click="deleteFund(props.item)">mdi-delete</v-icon></td>
            </tr>  
        </template>

      </v-data-table>
      <v-data-iterator 
                :items="funds"
                hide-default-footer
                v-else
              >
        <template v-slot:default="{ items, isExpanded, expand }">
                  <v-row>
                    <v-col
                      v-for="item in items"
                      :key="item.name"
                      cols="12"
                    >
                      <v-card>
                        <v-card-title class="pb-0 pt-0 pl-0">
                          <v-col cols="9" class="text-left body-2 text-truncate">{{item.cust_number}}</v-col>
                          <v-col cols="3" class="text-center">
                            <v-card-actions>
                              <v-icon @click="updateFund(item)" class="small">mdi-pencil</v-icon>
                              <v-icon @click="deleteFund(item)" class="small">mdi-delete</v-icon>
                              <v-icon @click.native="expand(item, !isExpanded(item))" class="small">mdi-dots-horizontal</v-icon>
                            </v-card-actions>
                          </v-col>
                        </v-card-title>
                        <v-divider></v-divider>

                        <v-list v-show="isExpanded(item)" dense>
                          <v-list-item>
                            <v-list-item-content>Customer:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.customer }}</v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Category:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.category }}</v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Fund Name:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.fund_name }}</v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Fund Acquired Value:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.fund_acquired_value }} </v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Fund Acquired Date:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.fund_acquired_date }} </v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Fund Recent Value:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.fund_recent_value }} </v-list-item-content>
                          </v-list-item>
                          <v-list-item>
                            <v-list-item-content>Fund Recent Date:</v-list-item-content>
                            <v-list-item-content class="align-end">{{ item.fund_recent_date }} </v-list-item-content>
                          </v-list-item>
                        </v-list>
                      </v-card>
                    </v-col>
                  </v-row>
                </template>     
              </v-data-iterator>  

      <v-btn class="blue white--text" @click="addNewFund">Add Fund</v-btn>
       </v-col>  
      </v-row>
    </v-container>
  </v-main>
</template>


<script>
  import router from '../router';
  import {APIService} from '../http/APIService';
  const apiService = new APIService();
  export default {
    name: "FundList",
    data: () => ({
      funds: [],
      validUserName: "Guest",
      fundSize: 0,
      showMsg: '',
      isMobile: false,
      headers: [
        
        {text: 'Customer Number', sortable: false, align: 'left',},
        {text: 'Category', sortable: false, align: 'left',},
        {text: 'Fund Name', sortable: false, align: 'left',},
        {text: 'Fund Acquired Value', sortable: false, align: 'left',},
        {text: 'Fund Acquired Date', sortable: false, align: 'left',},
        {text: 'Fund Recent Value', sortable: false, align: 'left',},
        {text: 'Fund Recent Date', sortable: false, align: 'left',},
        {text: 'Update', sortable: false, align: 'left',},
        {text: 'Delete', sortable: false, align: 'left',}
      ],
    }),
    mounted() {
      this.getFunds();
      this.showMessages();
    },
    methods: {

        onResize() {
          if (window.innerWidth < 600)
            this.isMobile = true;
          else  
            this.isMobile = false;
        },

      showMessages(){
        console.log(this.$route.params.msg)
         if (this.$route.params.msg) {
           this.showMsg = this.$route.params.msg;
         }
      },
      getFunds() {
        apiService.getFundList().then(response => {
          this.funds = response.data.data;
          this.fundSize = this.funds.length;
          if (localStorage.getItem("isAuthenticates")
            && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
            this.validUserName = JSON.parse(localStorage.getItem("log_user"));
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      },
      addNewFund() {
        if (localStorage.getItem("isAuthenticates")
          && JSON.parse(localStorage.getItem("isAuthenticates")) === true) {
          router.push('/fund-create');
        } else {
          router.push("/auth");
        }
      },
      updateFund(fund) {
        router.push('/fund-create/' + fund.pk);
      },
      deleteFund(fund) {
        apiService.deleteFund(fund.pk).then(response => {
          if (response.status === 204) {
           alert("Fund deleted");
           this.showMsg = 'deleted';
           router.push('/fund-list/deleted/');
           this.getFunds();
          }
        }).catch(error => {
          if (error.response.status === 401) {
            localStorage.removeItem('isAuthenticates');
            localStorage.removeItem('log_user');
            localStorage.removeItem('token');
            router.push("/auth");
          }
        });
      }
    }
  };
</script>