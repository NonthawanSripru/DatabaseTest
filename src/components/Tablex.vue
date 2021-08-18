<template>
  <v-card>
    <v-card-title>
      Test Data
      <v-spacer></v-spacer>
      <v-container fluid>
        <v-row>
          <v-col cols="12">
            <v-combobox
              v-model="select"
              :items="items"
              label="Choose Table"
              outlined
              dense
            ></v-combobox>
          </v-col>
        </v-row>
      </v-container>
    </v-card-title>
    <div>
      <v-data-table
        :headers="headers"
        :items="desserts"
        :options.sync="options"
        :server-items-length="totalDesserts"
        :loading="loading"
        class="elevation-1"
        :footer-props="{
          'items-per-page-options': [1, 3, 5],
        }"
      ></v-data-table>
    </div>
  </v-card>
</template>
<script>
const DW = "http://localhost:3000";
export default {
  data() {
    return {
      totalDesserts: 0,
      desserts: [],
      loading: true,
      options: {
        sortBy: [],
      },
      headers: [],
      select: ["Customers"],
      items: ["Customers", "Course"],
    };
  },
  watch: {
    options: {
      handler() {
        this.getDataFromApi();
      },
      deep: true,
    },
  },
   DataModel(val) {
      if (!val) {
        this.customers = [];
        this.headers = [];
      } else {
        this.getDataFromApi();
      }
    },
  mounted() {
    this.select = this.listDataModel();
    this.items = this.select[0];
    this.getDataFromApi();
  },
  methods: {
    listDataModel() {
      return [
        {
          Name: "Course",
          Function: "Get_Fact_Sales_and_Products",
          Params: "'Consumer'",
        },
      ];
    },
    async getDataFromApi() {
      this.loading = true;
      //let limited =2
      const { sortBy, sortDesc, page, itemsPerPage,select } = this.options;
      console.log(this.options);
      let xURL = DW;

      console.log(select);
      console.log(sortBy);
      

      if (!sortBy.length&&!select.length) {
        xURL = DW + `/page/${page}/limited/${itemsPerPage}`;
      } else if(!select.length) {
        xURL = DW + `/page/${page}/limited/${itemsPerPage}/table/${select}`;
      } else if(!sortBy.length){
        xURL = DW + `/page/${page}/limited/${itemsPerPage}/sort/${sortBy}`;
      } else {
        URL = DW + `/page/${page}/limited/${itemsPerPage}/table/${select}/sort/${sortBy}`;x
      }

      console.log(xURL);

      // let params =
      let rest = await axios.get(xURL);

      if (rest.data) {
        let items = rest.data;
        //console.log(items.value[0]);
        this.desserts = items.value;
        this.totalDesserts = items.total;
        let field = Object.keys(items.value[0]);
        this.headers = await field.map((e) => {
          return { text: e, value: e };
        });
        //const total = 4;
        this.loading = false;
      }
    },
  },
};
</script>