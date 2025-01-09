<template>
  <v-container fluid fill-height class="grey lighten-3">
    <v-slide-y-transition mode="out-in">
      <v-layout row wrap>
        <v-flex xs12 sm10 offset-sm1 md10 offest-md1 lg10 offset-lg1>
          <v-toolbar flat color="grey lighten-2">
            <v-toolbar-title>Advice-Surgery </v-toolbar-title>
            <v-divider class="mx-2 black" inset vertical></v-divider>
            <v-spacer></v-spacer>
            <v-autocomplete :items="branch" v-model="SetBranch" label="Branch:" item-text="shortCode" item-value="text"
              id="SelBranch"></v-autocomplete>

            <v-spacer></v-spacer>
            <v-autocomplete :items="AdviseType" v-model="SetAdvise" label="Type" item-text="shortCode"
              item-value="TEXT"></v-autocomplete>

              <v-spacer></v-spacer>
          <v-menu absolute ref="menu1" :close-on-content-click="false" v-model="menu1" :nudge-right="40" :return-value.sync="fromdate" lazy transition="scale-transition" offset-y full-width min-width="150px">
            <v-text-field slot="activator" v-model="fromdate" placeholder="Select From Date" prepend-inner-icon="event" readonly></v-text-field>
            <v-date-picker color="primary" v-model="fromdate" type="month" no-title scrollable :min="minDate" :max="maxDate" backgroundRevenue-color="red" style="box-shadow:none">
              <v-spacer></v-spacer>
              <v-btn flat color="primary" @click="menu1 = false" style="outline:none">Cancel</v-btn>
              <v-btn flat color="primary" @click="$refs.menu1.save(fromdate)" style="outline:none">Ok</v-btn>
            </v-date-picker>
          </v-menu>
          <v-btn rounded color="primary" dark @click="apiReqAdvSur(SetBranch,SetAdvise,fromdate)">Generate</v-btn>

          <download-excel :data="json_data" :fields="json_fields" type="csv" :name="fileName" :fetch="downloadExcel">
            <v-btn fab flat medium color="black">
              <v-tooltip bottom>
                <v-icon slot="activator" color="green darken-4">fas fa-file-excel</v-icon>
                <span>Export</span>
              </v-tooltip>
            </v-btn>
          </download-excel>
          </v-toolbar>
          <loading :active.sync="isLoading" :is-full-page="fullPage" color="#7f0000" loader="bars"></loading>

        </v-flex>
      </v-layout>
    </v-slide-y-transition>
  </v-container>

</template>
  
<script>
var curday = function(sp) {
  var today = new Date();
  var dd = today.getDate();
  var mm = today.getMonth() + 1; //As January is 0.
  var yyyy = today.getFullYear();

  if (dd < 10) dd = "0" + dd;
  if (mm < 10) mm = "0" + mm;
  return yyyy + sp + mm + sp + dd;
};
var start = function() {
  alert("Download Started... Please wait Untill Download complete....");
};
var stop = function() {
  alert("No Data Available. please generate the data");
};

export default {
  data: () => ({
    branch: null,
    SetBranch: null,
    SetAdvise: null,
    AdviseType: [
      {
        shortCode: "Cataract",
        text: "Cataract"
      },
      {
        shortCode: "Cornea",
        text: "Cornea"
      },
      {
        shortCode: "Refractive",
        text: "Refractive"
      },
      {
        shortCode: "VR Injection",
        text: "VR Injection"
      },
      {
        shortCode: "VR Surgery",
        text: "VR Surgery"
      },
      {
        shortCode: "Other Surgery",
        text: "other Surgery"
      },
      {
        shortCode: "Inv & Trt",
        text: "Inv & Trt"
      },
      {
        shortCode: "Optical",
        text: "Optical"
      },
      {
        shortCode: "Pharmacy",
        text: "Pharmacy"
      }
    ],
    minDate: "2022-04-01",
    maxDate: curday("-"),
    fromdate: null,
    menu1: false,
    isLoading: false,
    fullPage: true,
    AdviceData: null,
    json_data: null,
    json_meta: [
      {
        key: "charset",
        value: "utf-8"
      }
    ],
    json_fields: {
    
      BRANCH: "BRANCHCODE",
      PATIENTID: "PATIENT_MRN",
      PATIENTNAME: "PATIENTNAME",
      PATIENT_AGE: "AGE",
      PATIENT_GENDER:"GENDER",
      PATIENT_MOBILE:"PHONE",
      ADVISED_DATE:"ADVISED_DATE",
      CONSULTANT:"ENTERED_BY",
      PROCEDURE:"PROCEDURENAME",
      LATERALITY:"LATERALITY",
      BILLNO: "BILLNO",
      BILLDATE: "BILLDATE",
      SERVICENAME:"SERVICENAME"
    },
    fileName: null,
    fileDate: null,

  }),
  mounted() {
    this.loadbranch();
  },
  methods: {
    loadbranch() {
      let userid = JSON.parse(sessionStorage.getItem("normal_user"));
      this.isLoading = true;
      this.loading = true;
      this.axios
        .get(`${process.env.API_URL}/api-chbranch/${userid.userName}`)
        .then(response => {
          this.branch = response.data;
          this.isLoading = false;
        });
    },
    apiReqAdvSur(branch, advice, date) {
      if(branch==null||branch==''){
        alert("Please Select branch");
        return false;
      }
      if(advice==null||advice==''){
        alert("Please Select Advice");
        return false;
      }
      if(date==null||date==''){
        alert("Please Select Date");
        return false;
      }
      this.isLoading = true;
      const requestOptions = {
        headers: {
          "Content-Type": "application/json",
          token: `${process.env.API_KEY}`
        }
      };
//      this.axios
//        .post(
//          `${process.env.API_URL}/api-advsurgery`,
//          {
//            advDate: date,
//            advType: advice,
//            advBranch: branch
//          },
//          requestOptions
//        )
//        .then(response => {
//          this.isLoading = false;
//          if (response.status == 200) {
//            alert("Data Generated.. please Export the data")
//            this.processDataAdvSur(response.data);
//          } else {
//            alert("Error in loading data...");
//          }
//
//          var str = "_"
//        this.fileDate = date;
//        this.file = branch.concat(str,advice,str, this.fileDate)
//      
//
//        this.fileName = `Advice_surgery_${this.file}.csv`;
//
//        });
//    },
    processDataAdvSur(data) {
      this.AdviceData = data.result["data"];
    },
    downloadExcel() {
      let tempDataArr = [];
      console.log(this.AdviceData);
      if(this.AdviceData==null){
        stop()
      }
      else {
        start();
        if (this.fileDate !== null) {
        tempDataArr = this.AdviceData;
        for (let i = 0; i <= tempDataArr.length; i++) {
          if (i == tempDataArr.length) {
            alert("Download completed...");
          }
        }
        return tempDataArr;
      } else {
        alert("No data found");
        return null;
      }
      }
     

    }
  }
};
</script>
  
<style>
</style>
