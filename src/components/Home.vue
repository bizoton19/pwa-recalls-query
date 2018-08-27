<template>


  <v-container fill-height>
    <v-layout align-center>
      <v-flex>

        <div class="title mb-3">Check out our latest recalls</div>

        <ul>
          <li v-for="(r,index) in latestRecalls" v-bind:key="index">
            <v-card>
              <v-container fluid grid-list-md>
                <v-layout row wrap>
                  <v-flex v-for="img in r.images" v-bind="{ [`xs${index===1?12:6}`]: true }" :key="img.URL">
                    <v-card>
                      <v-card-media  :src="img.URL" height="200px">
                        <v-container  fluid pa-2>
                          <v-layout fill-height>
                            <v-flex xs12 align-end flexbox>
                              <span class="headline white--text" v-text="r.title"></span>
                            </v-flex>
                          </v-layout>
                        </v-container>
                      </v-card-media>

                      <v-card-actions>
                        <v-spacer></v-spacer>
                        <v-btn icon>
                          <v-icon>favorite</v-icon>
                        </v-btn>
                        <v-btn icon>
                          <v-icon>bookmark</v-icon>
                        </v-btn>
                        <v-btn icon>
                          <v-icon>share</v-icon>
                        </v-btn>
                      </v-card-actions>
                    </v-card>
                  </v-flex>
                </v-layout>
              </v-container>
            </v-card>
          </li>
        </ul>
        <div class="title mb-3">Check out our latest children recalls</div>

        <ul>
          <li v-for="(r,index) in childrenRecalls" v-bind:key="index">
            {{r.title}}
          </li>
        </ul>
      </v-flex>
    </v-layout>
  </v-container>



</template>
<script>
  import axios from "axios";
  import moment from "moment";
  export default {
    name: "home",
    data: function () {
      return {
        recalls: [],
        newRecalls: []
      };
    },
    computed: {
      latestRecalls: function () {
        return this.newRecalls;
      },
      childrenRecalls: function () {
        let vm = this;
        let cRecalls = [];
        let childrenKeyWords = [
          "children",
          "toddler",
          "baby",
          "toys",
          "infant",
          "kids"
        ];
        var recall = null;
        this.latestRecalls.forEach(r => {
          recall = r;
          childrenKeyWords.forEach(k => {
            console.log("recall is" + recall);
            console.log("recall desc" + recall.description);
            if (recall.description) {
              if (recall.description.includes(k)) {
                cRecalls.push(r);
              }
            }
          });
        });
        return cRecalls.slice(0, 10);
      }
    },
    created: function () {
      this.getLatestRecalls();
    },
    methods: {
      getLatestRecalls() {
        let vm = this;
        const cpscapi = process.env.ROOT_RECALLS_API;
        const apiRecallURL =
          "https://pwarecallsapiwrapper.azurewebsites.net/api/recalls/latest";
        const thirdwebsiteurl = window.location.href;
        const thirdwebsitetitle = document.title;
        vm.resultCount = 0;
        vm.resultCount = 0;
        vm.recalls = [];
        let mappedRequest = vm.mapRequestParams();
        console.log(mappedRequest);
        let requestParams = axios
          .get(apiRecallURL, {
            params: mappedRequest
          })
          .then(response => {
            if (response.data.length > 0) {
              vm.handleResponse(response);
            } else {
              //vm.showProgress = false;
              vm.newRecalls = [];
            }
          })
          .catch(error => {
            //vm.isError = true;
          });
      },
      mapRequestParams() {
        const vm = this;
        return {
          searchfor: vm.searchFor ? vm.searchFor : "",
          productname: vm.productName ? vm.productName : "",
          manufacturername: vm.manufacturer ? vm.manufacturer : "",
          producttype: vm.productType ? vm.productType : "",
          productModel: vm.productModel ? vm.productModel : "",
          recallDateEnd: moment().format("YYYY-MM-DD"),
          recallDateStart: moment(new Date(1970, 31, 12, 5, 0, 0)).format(
            "YYYY-MM-DD"
          ),
          thirdwebsiteurl: window.location.href
        };
      },
      handleResponse(response) {
        const vm = this;
        response.data.forEach(element => {
          console.log("handling");

          vm.newRecalls.push({
            title: element.Title,
            url: element.URL,
            recallDate: moment(element.RecallDate).format("MMM Do YYYY"),
            images: element.Images, //use array functions to filter
            description: element.Description,
            manufacturerCountries: element.ManufacturerCountries
          });
          vm.resultCount = vm.recalls.length;
        });
      }
    }
  };

</script>
