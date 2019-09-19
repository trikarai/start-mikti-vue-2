<template>
  <v-container>
    <notification :err_msg2="err_msg" :status2="status" />
    <!-- {{dataList.list}} -->
    <v-layout class="mb-3">
      <v-flex md3>
        <v-btn color="primary" @click="openIdeaForm()">
          <v-icon left>add</v-icon>
          {{ $vuetify.lang.t('$vuetify.action.add') }} {{ $vuetify.lang.t('$vuetify.idea.idea') }}
        </v-btn>
      </v-flex>
    </v-layout>
    <v-layout>
      <v-flex md12>
        <v-card class="elevation-0 mb-3">
          <v-card-title>
            <div class="flex-grow-1"></div>
            <v-text-field
              v-model="search"
              append-icon="search"
              label="Search"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
          <!-- {{dataList}} -->
          <v-card-text>
            <v-data-table
              :search="search"
              :loading="tableLoad"
              :headers="headers"
              :items="dataList.list"
              class="elevation-1"
            >
              <template v-slot:item.menu="{ item }">
                <div>
                  <v-btn class="ma-2 elevation-0" small fab color="primary">
                    <v-icon>zoom_in</v-icon>
                  </v-btn>
                </div>
              </template>
              <template v-slot:item.main="{ item }">
                <v-icon large v-if="item.is_main_idea" color="#FFDF00">star</v-icon>
                <v-btn small @click="setMain(item.id)" v-else>
                  <v-icon small>star</v-icon>
                </v-btn>
              </template>
              <template v-slot:item.action="{ item }">
                <v-btn small color="red" @click="deleteData(item.id)">
                  <v-icon small left>delete</v-icon>
                  {{$vuetify.lang.t('$vuetify.action.delete')}}
                </v-btn>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-flex>
    </v-layout>

    <idea-form v-model="dialogIdea" :edit="edit"></idea-form>
  </v-container>
</template>
<script>
import auth from "@/config/auth";
import * as config from "@/config/app.config";
import { statusMixins } from "@/mixins/statusMixins";
import { notifMixins } from "@/mixins/notifMixins";

import IdeaForm from "./IdeaForm";

export default {
  mixins: [statusMixins, notifMixins],
  components: {
    IdeaForm
  },
  data() {
    return {
      dialogIdea: false,
      edit: false,
      search: "",
      tableLoad: false,
      dataList: { total: 0, list: [] },
      headers: [
        {
          text: "",
          value: "menu",
          sortable: false
        },
        {
          text: "Name",
          value: "name",
          sortable: false
        },
        {
          text: "Elevator Pitch",
          value: "description",
          sortable: false
        },
        {
          text: "",
          value: "main",
          sortable: false
        },
        { text: "", value: "action", sortable: false, align: "right" }
      ]
    };
  },
  watch: {},
  created: function() {},
  mounted: function() {
    this.getDataList();
  },
  methods: {
    getDataList: function() {
      this.tableLoad = true;
      // this.resetNotif();
      this.axios
        .get(config.baseUri + "/team/" + this.$route.params.teamId + "/idea", {
          headers: auth.getAuthHeader()
        })
        .then(res => {
          if (res.data.data) {
            this.dataList = res.data.data;
          } else {
            this.dataList = { total: 0, list: [] };
          }
        })
        .catch(error => {
          this.showError(error);
        })
        .finally(() => {
          this.tableLoad = false;
        });
    },
    setMain(id) {
      this.tableLoad = true;
      this.axios
        .put(
          config.baseUri +
            "/team/" +
            this.$route.params.teamId +
            "/idea/" +
            id +
            "/assign_as_main_idea",
          {},
          { headers: auth.getAuthHeader() }
        )
        .then(res => {
          this.showSuccess(res, ["Success Set As Main Idea"]);
          this.getDataList();
        })
        .catch(error => {
          this.showError(error);
        })
        .finally(() => {
          this.tableLoad = false;
        });
    },
    deleteData(id) {
      this.tableLoad = true;
      this.axios
        .delete(
          config.baseUri + "/team/" + this.$route.params.teamId + "/idea/" + id,
          { headers: auth.getAuthHeader() }
        )
        .then(res => {
          this.showInfo(res, ["Selected Idea Deleted"]);
          this.getDataList();
        })
        .catch(error => {
          this.showError(error);
        })
        .finally(() => {
          this.tableLoad = false;
        });
    },
    gotoIdeaDetail(id) {
      this.$router.push({
        path: "/talent/team/" + this.$route.params.teamId + "/idea/" + id
      });
    },
    openIdeaForm() {
      // this.edit = true;
      this.dialogIdea = true;
    }
  }
};
</script>

<style scoped>
</style>