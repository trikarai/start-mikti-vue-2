<template>
  <v-container>
    <!-- {{dataList.list}} -->
    <v-row>
      <v-col cols="12" md="3">
        <v-btn color="primary" router to="/talent/team/create">
          <v-icon left>add</v-icon>
          {{ $vuetify.lang.t('$vuetify.action.create') }} {{ $vuetify.lang.t('$vuetify.team.team') }}
        </v-btn>
      </v-col>
      <v-col md="2" class="ml-2 mr-2">
        <v-select
          v-model="teamStatus"
          :label="'Status ' + $vuetify.lang.t('$vuetify.team.team')"
          :items="teamStatusList"
        ></v-select>
      </v-col>
      <v-col md="2" class="ml-2 mr-2">
        <v-select v-model="invStatus" :label="'Status Invitation'" :items="invStatusList"></v-select>
      </v-col>
    </v-row>
    <v-row>
      <v-col>
        <v-card class="elevation-0 mb-3">
          <!-- <v-card-title>
            <div class="flex-grow-1"></div>
            <v-text-field
              v-model="search"
              append-icon="search"
              label="Search"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>-->
          <!-- {{dataList}} -->
          <v-card-text>
            <v-data-table
              :search="search"
              :loading="tableLoad"
              :headers="headers"
              :items="dataList.list"
              class="elevation-1"
            >
              <template v-slot:item.teamName="{item}">{{item.team.name | truncate(20)}}</template>
              <template v-slot:item.creator="{item}">{{item.team.creator.name | truncate(20)}}</template>
              <template v-slot:item.status="{item}">
                <v-chip :color="getColor(item.status)" dark>{{ item.status }}</v-chip>
              </template>
              <template v-slot:item.action="{ item }">
                <template v-if="item.status == 'active'">
                  <v-btn small class="mr-2 mt-2" @click="gotoTeamDetail(item)">Manage</v-btn>
                  <v-btn small class="mr-2 mt-2" @click="gotoTeamIdea(item)">Idea</v-btn>
                  <v-btn small class="mr-2 mt-2" @click="gotoParticipation(item)">Program</v-btn>
                  <v-btn
                    small
                    class="mr-2 mt-2"
                    color="red"
                    @click="membershipAction(item.id, 'resign')"
                  >Resign</v-btn>
                </template>
                <template v-if="item.status == 'invited'">
                  <v-btn
                    small
                    class="mr-2 mt-2"
                    @click="membershipAction(item.id, 'accept_invitation')"
                  >Accept</v-btn>
                  <v-btn
                    small
                    class="mr-2 mt-2"
                    @click="membershipAction(item.id, 'refuse_invitation')"
                  >Reject</v-btn>
                </template>
              </template>
            </v-data-table>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import bus from "@/bus";
import auth from "@/config/auth";
import * as config from "@/config/app.config";
import { statusMixins } from "@/mixins/statusMixins";

export default {
  mixins: [statusMixins],
  data() {
    return {
      search: "",
      tableLoad: false,
      dataList: { total: 0, list: [] },
      headers: [
        { text: "Team", value: "teamName", sortable: false },
        {
          text: "Initiator",
          value: "creator",
          sortable: false
        },
        { text: "Region", value: "team.creator.region.name", sortable: false },
        { text: "Your Position", value: "position", sortable: false },
        { text: "Status", value: "status", sortable: false },
        { text: "", value: "action", sortable: false, align: "right" }
      ],
      invStatusList: [
        { value: "", text: "All" },
        { value: "invited", text: "Invited" }
      ],
      invStatus: "",
      teamStatusList: [
        { value: "", text: "All" },
        { value: "active", text: "Active" },
        { value: "resigned", text: "Resigned" }
      ],
      teamStatus: "active"
    };
  },
  watch: {
    teamStatus: "getDataList",
    invStatus: "getDataList"
  },
  created: function() {},
  mounted: function() {
    this.getDataList();
    // this.testNotif();
  },
  methods: {
    getDataList: function() {
      var status1, status2, join;
      if (this.teamStatus === "") {
        status1 = "";
      } else {
        status1 = "status[]=" + this.teamStatus;
      }

      if (this.teamStatus != "" && this.invStatus != "") {
        join = "&";
      } else {
        join = "";
      }

      if (this.invStatus === "") {
        status2 = "";
      } else {
        status2 = "status[]=" + this.invStatus;
      }

      this.tableLoad = true;
      this.axios
        .get(
          config.baseUri +
            "/talent/team/membership?" +
            encodeURI(status1) +
            join +
            encodeURI(status2),
          {
            headers: auth.getAuthHeader()
          }
        )
        .then(res => {
          if (res.data.data) {
            this.dataList = res.data.data;
          } else {
            this.dataList = { total: 0, list: [] };
          }
        })
        .catch(res => {
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.tableLoad = false;
        });
    },
    membershipAction(id, action) {
      this.tableLoad = true;
      this.axios
        .put(config.baseUri + "/talent/team/membership/" + id + "/" + action)
        .then(() => {
          this.getDataList();
        })
        .catch(res => {
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.tableLoad = false;
        });
    },
    gotoTeamDetail(item) {
      this.$router.push({
        path: "/talent/team/" + item.team.id + "/membership/" + item.id
      });
    },
    gotoTeamIdea(item) {
      this.$router.push({
        path: "/talent/team/" + item.team.id + "/idea"
      });
    },
    gotoParticipation(item) {
      this.$router.push({
        path: "/talent/team/" + item.team.id + "/participation"
      });
    }
  }
};
</script>

<style scoped>
</style>