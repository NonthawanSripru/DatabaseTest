<template>
  <v-card>
    <v-card-title class="white--text blue-grey">
      Test Data
      <v-spacer/> 
      <v-combobox
        v-model="xTable"
        :items="['Customers','Employees']"
        label="Choose Table"
        outlined
        dense
      ></v-combobox>
    </v-card-title>

    <v-data-table
      :headers="headers"
      :items="desserts"
      :options.sync="options"
      :server-items-length="totalDesserts"
      :loading="loading"
      class="elevation-1"
      :footer-props="{
        'items-per-page-options': [5, 10, -1],
      }"
    ></v-data-table>
  </v-card>
</template>
<script>
const DW = "http://localhost:3000";
export default {
  data() {
    return {
      totalDesserts: 0,
      desserts: [],
      xTable:"Customers",
      loading: true,
      options: {
        sortBy: [],
      },
      headers: [],
    };
  },
  watch: {
    xTable(){
      this.options = { sortBy: [], sortDesc: [], page: 1, itemsPerPage: 10 }
       this.getDataFromApi();
    },
    options: {
      handler() {
        this.getDataFromApi();
      },
      deep: true,
    },
  },
  mounted() {
    this.getDataFromApi();
  },
  methods: {
    async getDataFromApi() {
      this.loading = true;
      let xURL = DW+`/table/${this.xTable}`;
      const { sortBy, sortDesc, page, itemsPerPage } = this.options;
      let xop = {
        sortBy,
        sortDesc,
        page,
        itemsPerPage,
      };
      if (itemsPerPage == -1) {
        xop.itemsPerPage = this.totalDesserts;
      }

      let rest = await axios.post(xURL, xop);
      if (rest.data) {
        let items = rest.data;
        this.desserts = items.value;
        this.totalDesserts = items.total;
        let field = Object.keys(items.value[0]);
        this.headers = await field.map((e) => {
          return { text: e, value: e };
        });
        this.loading = false;
      }
    },
  },
};
</script>