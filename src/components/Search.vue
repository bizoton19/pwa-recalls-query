<template >

<v-container >
        <v-layout   wrap>

      <!--<v-flex xs12 sm12 md4 lg4 xl4>-->
       <div id="search">
        <v-form   ref="form"  >
    <v-text-field
      prepend-icon="search"
      color="green darken-2"
      label="Search For"
      v-model="searchFor"
      clearable
      @keyup.enter.tab="submit()"
    >
    </v-text-field>
    <v-text-field
    prepend-icon="title"
     color="purple darken-2"
      label="Product Name"
      clearable
      v-model="productName"
      @keyup.enter.tab="submit()"
    ></v-text-field>
    
    <v-select
    v-show="hasResult"
     prepend-icon="business"
    
      :disabled="formState.cleared && !manufacturers.length>0"
      label="Manufacture"
      v-model="manufacturer"
      :items="manufacturers"
      clearable
      
    ></v-select>
    <v-select
    
    v-show="hasResult"
    :disabled="formState.cleared && !productTypes.length>0"
    prepend-icon="battery_unknown"
      label="Product Type"
      v-model="productType"
      :items="productTypes"
      clearable
    ></v-select>
    <v-text-field
     prepend-icon="battery_unknown"
      label="Product Model"
      v-model.trim="productModel"
      :value="productModel"
    ></v-text-field>
    <v-select
     prepend-icon="date_range"
     color="orange darken-2"
      label="Date Range"
      v-model="relativeDate"
      :items="relativeDates"
      :value="relativeDate"
      clearable
    ></v-select>

    <v-btn  small round  @click.prevent="clear" color="indigo darken-1" dark   >
      Reset
      <v-icon color="orange lighten-2" right>refresh</v-icon>
      </v-btn>
      <v-btn
      small
      round
      color="indigo darken-1"
      class="white--text"
      :loading="showProgress && !hasResult && !isError"
      @click.prevent.stop="isAtLeastOneFieldValid?submit():showFormValidDialog=true"
      :disabled="showProgress && !hasResult && !isError"
      
    >
      Apply
       <v-icon color="green lighten-2" light right>check_circle</v-icon>
      
    </v-btn>
    
  </v-form>
  </div>
   <v-alert v-if="!hasResult && formState.completed" color="grey" icon="info" value="true">
      No records found
    </v-alert>
    <v-alert v-else-if=" resultCount > 0 && formState.completed" color="grey" icon="info" value="true">
      {{resultCount}} record(s) found
    </v-alert>

    <!--</v-flex>-->
   
   
   <v-flex v-if="isError">
    <div>
    <v-alert color="error" icon="warning" value="true">
      Error communicating with the back end service, please try again later
    </v-alert>
  </div>
  
    </v-flex>
    
   <!-- <result-list v-if="hasResult" :recalls="recalls"></result-list>
    -->
     


 <v-dialog v-model="showFormValidDialog" max-width="500px">
        <v-card>
          <v-card-title>
            <span><v-icon left medium >block</v-icon>Validation</span>
            <v-spacer></v-spacer>
            
          </v-card-title>
          
          <v-card-text>  At least one field is required in order to initiate a search</v-card-text>
          <v-card-actions>
            <v-btn color="primary" flat @click.stop="showFormValidDialog=false">Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>
      </v-container>
</template>

<script>
//import resultList from "./ResultList.vue";
import axios from "axios";
import moment from "moment";
import { eventBus, EventBus } from "../eventBus.js";
export default {
  name: "search",
  components: {
    //resultList
  },
  data: function() {
    return {
      showProgress: false,
      formState: {
        started: false,
        cleared: true,
        completed: false
      },

      loader: null,
      searchFor: "",
      productName: "",
      manufacturer: "",
      productType: "",
      productModel: "",
      relativeDate: "",
      productType: "",
      recallDates: {
        recallStartDate: "",
        recallEndDate: ""
      },
      cpscRequest: {},

      manufacturers: [],
      productTypes: [],
      relativeDates: [
        { value: 0, text: "All Dates" },
        { value: 3, text: "Last 3 Months" },
        { value: 6, text: "Last 6 Months" },
        { value: 9, text: "Last 9 Months" },
        { value: 12, text: "Last 12 Months" }
      ],
      recalls: [
        {
          title: "",
          recallDate: "",
          Images: [],
          url: ""
        }
      ],
      thirdwebsiteurl: "",
      showFormValidDialog: false,
      resultCount: 0,
      isError: false
    };
  },
  mounted: function() {
    console.log("has result", this.hasResult);
  },
  computed: {
    hasResult: function() {
      return this.resultCount > 0 ? true : false;
    },

    isAtLeastOneFieldValid: function() {
      const vm = this;

      return (
        (vm.searchFor !== undefined && vm.searchFor !== "") ||
        (vm.productName !== undefined && vm.productName !== "") ||
        (vm.manufacturer !== undefined && vm.manufacturer !== "") ||
        (vm.productModel !== undefined && vm.productModel !== "") ||
        (vm.productType !== undefined && vm.productType !== "") ||
        (vm.relativeDate !== undefined && vm.relativeDate !== "")
      );
    }
  },
  watch: {
    relativeDate: function(number) {
      this.setDateRange(number);
    }
  },
  props: {
    source: String
  },
  created: function() {
    let vm = this;
  },
  methods: {
    setDateRange(number) {
      let vm = this;
      if (number > 0) {
        vm.recallDates.recallEndDate = moment().format("YYYY-MM-DD");
        vm.recallDates.recallStartDate = moment()
          .subtract(number, "months")
          .format("YYYY-MM-DD");
      } else {
        vm.recallDates.recallEndDate = "";
        vm.recallDates.recallStartDate = "";
      }
    },
    submit() {
      let vm = this;
      const cpscapi = process.env.ROOT_RECALLS_API;
      const apiRecallURL = cpscapi + "api/Recall/GetRecall";
      const thirdwebsiteurl = window.location.href;
      const thirdwebsitetitle = document.title;
      vm.resultCount = 0;
      vm.formState.completed = false;
      let isvalid = vm.isAtLeastOneFieldValid;
      if (isvalid) {
        vm.isError = false;
        vm.showProgress = true;
        vm.resultCount = 0;
        vm.formState.cleared = false;
        vm.formState.started = true;
        vm.recalls = [];
        let mappedRequest = vm.mapRequestParams();

        let requestParams = axios
          .get(apiRecallURL, {
            params: mappedRequest
          })
          .then(response => {
            if (response.data.length > 0) {
              vm.handleResponse(response);
              EventBus.$emit("searchResultFetched", {
                resultCount: vm.resultCount,
                recalls: vm.recalls
              });
              
            } else {
              vm.showProgress = false;
              vm.formState.started = false;
              vm.formState.completed = true;
              vm.recalls = [];
            }
          })
          .catch(error => {
            vm.isError = true;
            vm.formState.completed = true;
            console.log(error);
          });
          this.$router.push("resultList");//show resultlist route
      }
    },
    mapRequestParams() {
      const vm = this;
      return {
        searchfor: vm.searchFor ? vm.searchFor : "",
        productname: vm.productName ? vm.productName : "",
        manufacturername: vm.manufacturer ? vm.manufacturer : "",
        producttype: vm.productType ? vm.productType : "",
        productModel: vm.productModel ? vm.productModel : "",
        recallDateEnd: vm.relativeDate > 0 ? vm.recallDates.recallEndDate : "",
        recallDateStart:
          vm.relativeDate > 0 ? vm.recallDates.recallStartDate : "",
        dateRange: vm.relativeDate ? vm.relativeDate : "",
        thirdwebsiteurl: window.location.href
      };
    },
    handleResponse(response) {
      const vm = this;
      response.data.forEach(element => {
        element.Manufacturers.forEach(m => {
          vm.manufacturers.push(m.Name);
        });
        element.Products.forEach(p => {
          vm.productTypes.push(p.Type);
        });
        vm.recalls.push({
          title: element.Title,
          url: element.URL,
          recallDate: moment(element.RecallDate).format("MMM Do YYYY"),
          images: element.Images, //use array functions to filter
          description: element.Description,
          products: element.Products,
          injuries: element.Injuries,
          manufacturers: element.Manufacturers,
          manufacturerCountries: element.ManufacturerCountries,
          productUpcs: element.ProductUPCs,
          hazards: element.Hazards,
          remedies: element.Remedies,
          retailers: element.Retailers
        });
        vm.resultCount = vm.recalls.length;
      });

      vm.formState.completed = true;
    },
    clear() {
      this.$refs.form.reset();
      this.resultCount = 0;
      this.showProgress = false;
      this.isError = false;
      this.formState.started = false;
      this.formState.completed = false;
      this.formState.cleared = true;
      EventBus.$emit("formCleared", true);
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.spinner {
  text-align: center;
  margin-top: 50%;
}
.progress-circular {
  margin: 1rem;
}
</style>
