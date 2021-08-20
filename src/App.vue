<template>
  <v-app v-if="$route.name == 'admin'">
    <router-view />
  </v-app>
  <v-app v-else>
    <v-app-bar app color="white" flat>
      <v-row align="center" justify="space-around">
        <v-container class="py-0 fill-height">
          <v-avatar size="50">
            <v-img :src="img.VSTECSLOGO" />
          </v-avatar>
          <h1 class="blue--text text--darken-4">VSTECS</h1>
          <v-spacer />
          <v-col v-for="link in links" :key="link">
            <v-btn
              @click="clickmenu(link)"
              depressed
              block
              color="blue-grey"
              dark
              x-large
            >
              {{ link }}
            </v-btn>
          </v-col>
          <v-spacer></v-spacer>
          {{ user.name }}
          <!-- <v-btn depressed color="primary" @click="xlogout"> Click </v-btn> --> 
        </v-container>
      </v-row>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
    <v-footer app color="blue-grey" class="white--text">
      <span>VST ECS (Thailand) Co.,Ltd.</span>
      <v-spacer />
      <span>&copy; 2021 (version {{ version }})</span>
    </v-footer>
  </v-app>
</template>

 
<script>
import { msalMixin } from "vue-msal";
const axios = require("axios").default;
const vAPI = process.env.VUE_APP_API;
const vAppData = vAPI + process.env.VUE_APP_DATA;
const VERSTION = process.env.VUE_APP_VERSION;
export default {
  mixins: [msalMixin],
  methods: {},
  data() {
    return {
      img: {
        VSTECSLOGO: "",
      },
      AccessToken: null,
      links: null,
      version: VERSTION,
    };
  },
  created() {
    this.init();
  },
  methods: {
    async xlogout() {
      const calldata = axios.create({
        baseURL: "http://localhost:3000",
        timeout: 1000,
        headers: { "Authorization": `Bearer ${this.AccessToken}` },
      });
      calldata
        .get("http://localhost:3000")
        .then((e) => console.log(e));
    },
    async init() {
      this.img.VSTECSLOGO = await this.GetImg("VSTECSLOGO");
      this.links = await this.GetData("TOPMENU");
    },
    async GetImg(val) {
      let xURL = vAppData + val;
      let rest = await axios.get(xURL);
      return "data:image/jpeg;base64," + rest.data[0].value;
    },
    async GetData(val) {
      let xURL = vAppData + val;
      let rest = await axios.get(xURL);
      return rest.data[0].value;
    },
    async clickmenu(item) {
      if (item != this.$route.name) {
        this.$router.push({ name: item }).go;
      }
    },
  },
  computed: {
    user() {
      let user = {};
      if (this.msal.isAuthenticated) {
        user = {
          ...this.msal.user,
          profile: {},
        };
        // console.log(user);
        this.$msal
          .acquireToken()
          .then((e) => (this.AccessToken = e.accessToken));
        if (this.msal.graph && this.msal.graph.profile) {
          user.profile = this.msal.graph.profile;
        }
      } else {
        this.$msal.signIn();
      }
      return user;
    },
  },
};
</script>