<template>
  <v-card class="mx-auto">
    <v-container fluid>
      <v-row>
        <v-col cols="12">
          <v-card>
            <v-card-title class="white--text blue-grey">
              <v-app-bar-nav-icon
                @click="configmodel = true"
              ></v-app-bar-nav-icon>
              Data Sample
              <v-spacer></v-spacer>
              <v-combobox
                dark
                v-model="DataModel"
                :items="DataModelList"
                item-text="Name"
                DataModelList
                clearable
                hide-selected
              ></v-combobox>
            </v-card-title>
            <v-data-table
              :headers="headers"
              :items="desserts"
              :options.sync="options"
              :server-items-length="totalDesserts"
              :loading="loading"
              :footer-props="{
                'items-per-page-options': [5, 10, 50, 100, 200, 500],
              }"
              class="elevation-1"
            ></v-data-table>
          </v-card>
        </v-col>
      </v-row>
      <v-dialog v-model="configmodel" max-width="600">
        <v-expansion-panels focusable v-model="panelmodel">
          <v-expansion-panel>
            <v-expansion-panel-header class="white--text blue-grey">
              Mart Select
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-combobox
                v-model="DataModel"
                :items="DataModelList"
                item-text="Name"
                DataModelList
                clearable
                hide-selected
              ></v-combobox>
            </v-expansion-panel-content>
          </v-expansion-panel>
          <v-expansion-panel>
            <v-expansion-panel-header class="white--text blue-grey">
              Configure Filter
            </v-expansion-panel-header>
            <v-expansion-panel-content> 

              <v-container fluid>
                <v-row>
                  <v-col cols="12">
                    <v-combobox
                      v-model="querys"
                      :items="yearQuery"
                      label="Filter Options"
                      multiple
                      outlined
                      dense
                    ></v-combobox>
                  </v-col>
                </v-row>
              </v-container>
            </v-expansion-panel-content>

          </v-expansion-panel>
          <v-expansion-panel>
            <v-expansion-panel-header class="white--text blue-grey">
              Configure fieldset
            </v-expansion-panel-header>
            <v-expansion-panel-content>
              <v-combobox
                v-model="headers"
                :items="fieldlist"
                label="Select Field Set"
                multiple
                chips
                hide-selected
              >
                <template v-slot:selection="{ attrs, item, parent, selected }">
                  <v-chip
                    v-if="item === Object(item)"
                    v-bind="attrs"
                    :color="`${item.color} lighten-3`"
                    :input-value="selected"
                    label
                    :filter="filter"
                    small
                  >
                    <span class="pr-2">
                      {{ item.text }}
                    </span>
                    <v-icon small @click="parent.selectItem(item)">
                      $delete
                    </v-icon>
                  </v-chip>
                </template>
              </v-combobox>
            </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-dialog>
    </v-container>
  </v-card>
</template>
 
<script>
const DW = process.env.VUE_APP_WAREHOUSE;
export default {
  data() {
    return {
      configmodel: false,
      DataModel: {},
      DataModelList: [],
      panelmodel: 0,
      totalDesserts: 0,
      desserts: [],
      loading: true,
      options: {
        page: 1,
        itemsPerPage: 3,
        sortBy: [],
        sortDesc: [true],
        groupBy: [],
        groupDesc: [],
        mustSort: false,
        multiSort: false,
      },
      headers: [],
      fieldlist: [],
      query: [],
      yearsQuery:['2021','2020','2019'],
    };
  },
  watch: {
    DataModel(val) {
      if (!val) {
        this.desserts = [];
        this.fieldlist = [];
        this.headers = [];
        this.querys =[];
      } else {
        this.getDataFromApi();
      }
    },
    options: {
      handler() {
        this.getDataFromApi();
      },
      deep: true,
    },
  },
  mounted() {
    this.DataModelList = this.listDataModel();
    this.DataModel = this.DataModelList[0];
    this.getDataFromApi();
  },
  methods: {
    listDataModel() {
      return [
        {
          Name: "Fact_Sales_and_Products",
          Function: "Get_Fact_Sales_and_Products",
          Params: "'Consumer'",
        },
      ];
    },
    filter(item, queryText, itemText) {
      if (item.header) return false;

      const hasValue = (val) => (val != null ? val : "");

      const text = hasValue(itemText);
      const query = hasValue(queryText);

      return (
        text.toString().toLowerCase().indexOf(query.toString().toLowerCase()) >
        -1
      );
    },
    async getDataFromApi() {
      this.loading = true;
      const { sortBy, sortDesc, page, itemsPerPage } = this.options;
      //console.log(this.options);

      let xURL =
        DW +
        `/function/${this.DataModel.Function}/params/${this.DataModel.Params}`;

      let rest = await axios.post(xURL, this.options);
      //console.log(rest);
      let items = rest.data.value;
      let field = Object.keys(items[0]).filter((e) => e != "Num");

      //let field = Object.keys(items[0])
      const total = rest.data.TableStatus.dataset_statistics[0].table_size;
      this.headers = await field.map((e) => {
        return { text: e, value: e };
      });
      this.fieldlist = this.headers;
      this.desserts = items;
      this.totalDesserts = total;
      this.loading = false;
    },
  },
};
</script>