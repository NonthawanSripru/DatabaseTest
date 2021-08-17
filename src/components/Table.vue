<template>
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
  mounted() {
    this.getDataFromApi();
  },
  methods: {
    async getDataFromApi() {
      this.loading = true;
      //let limited =2
      const { sortBy, sortDesc, page, itemsPerPage } = this.options;
      console.log(this.options);
      let xURL = DW 

      if(!sortBy.length){
         xURL = DW + `/page/${page}/limited/${itemsPerPage}`
      }else {
        xURL = DW + `/page/${page}/limited/${itemsPerPage}/sort/${sortBy}`
      }
       

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