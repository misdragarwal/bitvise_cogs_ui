<template>
    <v-container fluid fill-height class="grey lighten-3">
        <v-slide-y-transition mode="out-in">
            <v-layout row wrap>
                <v-flex xs12 sm10 offset-sm1 md10 offest-md1 lg10 offset-lg1>
                    <v-toolbar flat color="grey lighten-2">
                        >
                        <v-toolbar-title> DRT doctors cut off date </v-toolbar-title>
                        <v-spacer></v-spacer>
                    </v-toolbar>
                    <loading :active.sync="isLoading" :is-full-page="fullPage" color="#7f0000" loader="bars"></loading>
                    <template>
                        <v-card-title>
                            <th width="50%">
                                <v-autocomplete :items="comp_branch" v-model="compBranch" multiple
                                    label="Select branches" item-text="shortCode" item-value="TEXT">
                                </v-autocomplete>
                            </th>
                            <v-spacer></v-spacer>
                            <v-menu absolute ref="menu3" v-if="username=='finadmin'" :close-on-content-click="false"
                                v-model="menu3" :nudge-right="40" :return-value.sync="fromdate1" lazy
                                transition="scale-transition" offset-y full-width min-width="150px">
                                <v-text-field slot="activator" v-model="fromdate1" placeholder="Select Date"
                                    prepend-inner-icon="event" readonly></v-text-field>
                                <v-date-picker color="primary" v-model="fromdate1" no-title scrollable :min="minDate1"
                                    :max="maxDate" backgroundRevenue-color="red" style="box-shadow:none">
                                    <v-spacer></v-spacer>
                                    <v-btn flat color="primary" @click="menu3 = false" style="outline:none">Cancel
                                    </v-btn>
                                    <v-btn flat color="primary" @click="$refs.menu3.save(fromdate1)"
                                        style="outline:none">Ok</v-btn>
                                </v-date-picker>
                            </v-menu>

                            <v-btn color="primary" v-if="username=='finadmin'" dark
                                @click="lockdate(fromdate1,compBranch)">
                                Lockdate
                            </v-btn>
                        </v-card-title>

                        <v-card-text>
                            <table class="table table-hover table-bordered" v-if="show">
                                <thead>
                                    <tr class="grey lighten-2">
                                        <th class="text-xs-left" scope="col">Branch</th>
                                        <th class="text-xs-center" scope="col">Fixed date</th>
                                        <th class="text-xs-center" scope="col">Updated at</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-for="item in drtbranchdetail" :key="item.branch" scope="row"
                                        :class="'font-weight-black indigo lighten-2'">

                                        <td scope="row" class="text-xs-center">{{item.branch}}</td>
                                        <td scope="row" class="text-xs-center">{{item.fixdate}}</td>
                                        <td scope="row" class="text-xs-center">{{item.inserted_at}}</td>
                                    </tr>


                                </tbody>
                            </table>

                        </v-card-text>

                    </template>
                </v-flex>
            </v-layout>

        </v-slide-y-transition>
    </v-container>

</template>

<script>
var curday = function (sp) {
    var today = new Date();
    var dd = today.getDate();
    var mm = today.getMonth() + 1; //As January is 0.
    var yyyy = today.getFullYear();

    if (dd < 10) dd = "0" + dd;
    if (mm < 10) mm = "0" + mm;
    return yyyy + sp + mm + sp + dd;
};

export default {
    data: () => ({
        comp_branch: [],
        compBranch: [],
        menu3: false,
        username: "",
        fromdate1: null,
        minDate1: "2020-04-01",
        maxDate: curday("-"),
        isLoading: false,
        fullPage: true,
        drtbranchdetail: null,
        show: false
    }),
    mounted() {
        this.loadbranch();
        this.loadbranchdate()
    },
    methods: {
        loadbranch() {
            let userid = JSON.parse(sessionStorage.getItem("fin_user"));
            this.axios
                .get(`${process.env.API_URL}/api-finbranch/${userid.userName}`)
                .then(response => {
                    this.comp_branch = response.data;
                    this.username = userid.name;
                });
        },
        lockdate(data, branch) {
          this.isLoading = true;
            const requestOptions = {
                headers: {
                    "Content-Type": "application/json",
                    token: `${process.env.API_KEY}`
                }
            };
            this.axios.post(`${process.env.API_URL}/api-drtfixdate`, {
                date: data,
                branch: branch
            }, requestOptions).then(response => {
                if (response.data.code == 200) {
                    alert("Updated successfully")
                    this.compBranch = ''
                    this.loadbranch()
                    this.loadbranchdate()
                    this.isLoading = false;
        
                }

            })


        },
        loadbranchdate() {
            const requestOptions = {
                headers: {
                    "Content-Type": "application/json",
                    token: `${process.env.API_KEY}`
                }
            };
            this.axios.get(`${process.env.API_URL}/api-drtfixbranchdate`, requestOptions).then(response => {
                console.log(response);
                this.drtbranchdetail = response.data.data
                this.show = true;
            })
        }
    }
};
</script>

<style>

</style>
