<template>
  <v-container>
    <notification ref="notif" :err_msg2="err_msg" :status2="status" />

    <v-layout>
      <!-- {{dataTemplate}} -->
      <v-flex md12>
        <v-card>
          <v-toolbar flat color="primary" dark>
            <v-toolbar-title>{{$vuetify.lang.t('$vuetify.idea.experiment')}}</v-toolbar-title>
          </v-toolbar>
          <v-tabs v-model="tab" background-color="sidebar">
            <v-tabs-slider></v-tabs-slider>

            <v-progress-linear v-if="showTemplate" indeterminate color="green"></v-progress-linear>
            <template v-else>
              <v-tab href="#tab-javelin">Javelin Board</v-tab>
              <template v-if="dataTemplate.total != 0">
                <v-tab
                  @click="getExpData(dataTemplate.list[3].id)"
                  href="#tab-3"
                >{{dataTemplate.list[3].name}}</v-tab>
                <v-tab
                  @click="getExpData(dataTemplate.list[0].id)"
                  href="#tab-0"
                >{{dataTemplate.list[0].name}}</v-tab>
                <v-tab
                  @click="getExpData(dataTemplate.list[1].id)"
                  href="#tab-1"
                >{{dataTemplate.list[1].name}}</v-tab>
                <v-tab
                  @click="getExpData(dataTemplate.list[2].id)"
                  href="#tab-2"
                >{{dataTemplate.list[2].name}}</v-tab>
              </template>
            </template>
          </v-tabs>

          <v-tabs-items v-model="tab">
            <v-tab-item value="tab-javelin">
              <v-container>
                <v-row no-gutters>
                  <v-col>
                    <v-btn small class="ma-3 mt-5" color="accent" @click="loadSummary">
                      <v-icon left small>refresh</v-icon>Load Javelin Data
                    </v-btn>
                  </v-col>
                  <v-col md="6">
                    <v-select
                      label="Type"
                      v-model="exptype"
                      class="ma-2"
                      :items="expitems"
                      item-value="value"
                      item-text="text"
                    ></v-select>
                  </v-col>
                </v-row>
                <v-fade-transition>
                  <v-row no-gutters align="center" v-if="!integrityCheck">
                    <v-alert dense outlined type="error">
                      <strong>Data Integrity Failure</strong> : Please Update/Delete Obsolete Experiment Form Data
                    </v-alert>
                  </v-row>
                </v-fade-transition>

                <v-row v-if="!summary">
                  <v-progress-circular class="ma-5" size="50" color="primary" indeterminate></v-progress-circular>
                </v-row>
                <v-scale-transition>
                  <v-row v-if="dataJavelin.total > 4">
                    <v-alert
                      class="ma-3"
                      dense
                      type="info"
                      elevation="0"
                    >Tips : use of shift + scroll wheel to scroll horizontally</v-alert>
                  </v-row>
                </v-scale-transition>
              </v-container>
              <v-container style="overflow-x: scroll;">
                <v-scale-transition>
                  <v-row v-if="dataJavelin.total != 0">
                    <!-- <v-simple-table>
                    <template v-slot:default>-->
                    <table id="javeboard" style="table-layout: fixed; width: 100%;">
                      <tr>
                        <th style="padding-left:20px;padding-right:20px;"></th>
                        <th
                          style="background:#0085a4;text-align:center;padding-top:10px;padding-bottom:10px;padding-left:20px;padding-right:20px;"
                          v-for="javelin in reOrderDate(dataJavelin.list)"
                          :key="javelin.id"
                        >
                          <v-card flat color="#0085a4">
                            <v-card-title>{{javelin.name}}</v-card-title>
                            <v-card-actions>
                              <div class="flex-grow-1"></div>
                              <v-btn
                                x-small
                                fab
                                color="primary"
                                @click="editBsForm(javelin, javelin.name)"
                              >
                                <v-icon small>edit</v-icon>
                              </v-btn>
                              <v-btn x-small fab color="accent">
                                <v-icon small>zoom_in</v-icon>
                              </v-btn>
                            </v-card-actions>
                          </v-card>
                        </th>
                      </tr>
                      <tr>
                        <th class="jbth">Tanggal</th>
                        <td width="200px" v-for="javelin in reOrderDate(dataJavelin.list)" :key="javelin.id">
                          <span
                            style="color:#0085a4;font-weight:900;"
                          >{{javelin.date | moment("dddd")}}</span>
                          <br />
                          {{javelin.date | moment("MMMM")}}
                          <br />
                          {{javelin.date | moment("Do YYYY")}}
                        </td>
                      </tr>
                      <template v-for="i in 9">
                        <tr :key="i">
                          <th
                            class="jbth"
                          >{{reOrderField(dataJavelin.list[0].fields)[i].field_template.name}}</th>
                          <td
                            v-if="reOrderField(javelin.fields)[i].hasOwnProperty('selected_options')"
                            v-for="javelin in reOrderDate(dataJavelin.list)" :key="javelin.id"
                          >
                            <template
                              v-if="reOrderField(javelin.fields)[i].selected_options.length != 0"
                            >
                              <template
                                v-for="opt in reOrderField(javelin.fields)[i].selected_options"
                              >
                                <v-icon
                                  :key="opt.id"
                                  left
                                  v-if="opt.option.name == 'Pivot'"
                                >call_split</v-icon>
                                <v-icon
                                  :key="opt.id"
                                  left
                                  v-if="opt.option.name == 'Persevere'"
                                >lock</v-icon>
                                {{opt.option.name}}
                              </template>
                            </template>
                          </td>
                          <td v-else>{{reOrderField(javelin.fields)[i].value}}</td>
                        </tr>
                      </template>
                    </table>

                    <!-- </template>
                    </v-simple-table>-->
                  </v-row>
                </v-scale-transition>
              </v-container>
            </v-tab-item>
            <v-tab-item
              v-for="(data, index) in dataTemplate.list"
              :value="'tab-'+index"
              :key="data.id"
            >
              <template v-if="loadExpData">
                <v-progress-circular class="ma-5" size="50" color="primary" indeterminate></v-progress-circular>
              </template>
              <template v-else>
                <v-container>
                  <v-row>
                    <v-flex md12>
                      <v-btn
                        color="primary"
                        small
                        class="ma-2"
                        @click="openBsForm(data.id, data.name)"
                      >
                        <v-icon>add</v-icon>
                      </v-btn>
                    </v-flex>
                    <v-flex md12>
                      <v-data-table class="ma-2" :headers="headers" :items="dataExp.list">
                        <template v-slot:item.action="{ item }">
                          <v-btn color="accent" small class="ma-1" @click="gotoExpDetails(item.id)">
                            <v-icon small>zoom_in</v-icon>
                          </v-btn>
                          <v-btn
                            @click="editBsForm(item, item.name)"
                            color="primary"
                            small
                            class="ma-1"
                          >
                            <v-icon small>edit</v-icon>
                          </v-btn>

                          <v-btn
                            :loading="loadDelete"
                            @click="deleteAct(item.id, item.experiment_template.id)"
                            color="warning"
                            small
                            class="ma-1"
                          >
                            <v-icon small>delete</v-icon>
                          </v-btn>
                        </template>
                      </v-data-table>
                    </v-flex>
                  </v-row>
                </v-container>
              </template>
            </v-tab-item>
          </v-tabs-items>
        </v-card>
      </v-flex>
    </v-layout>

    <v-dialog v-model="dialogDelete" width="300">
      <v-card>
        <v-card-title>Delete Idea</v-card-title>
        <v-card-text>Are you sure want to Delete this Idea ?!</v-card-text>
        <v-card-actions>
          <div class="flex-grow-1"></div>
          <v-btn
            @click="deleteData(deleteId, templateId)"
            color="red"
          >{{$vuetify.lang.t('$vuetify.action.yes')}}</v-btn>
          <v-btn @click="dialogDelete = false">{{$vuetify.lang.t('$vuetify.action.cancel')}}</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <experiment-form
      v-if="dialogCanvas"
      :edit="edit"
      :template="templateData"
      :templateId="templateId"
      :templateName="templateName"
      @close="dialogCanvas = false"
      @refresh="refreshCanvas"
    ></experiment-form>
  </v-container>
</template>
<script>
import auth from "@/config/auth";
import bus from "@/bus";
import * as config from "@/config/app.config";
import { experimentMixins } from "@/mixins/experimentMixins";
import { getexpMixins } from "@/mixins/getexpMixins";

import ExperimentForm from "../ExperimentForm";

export default {
  mixins: [getexpMixins, experimentMixins],
  components: {
    ExperimentForm
  },
  data() {
    return {
      dialogCanvas: false,
      edit: false,
      showTemplate: false,
      templateName: "",
      dataTemplate: { total: 0, list: [] },
      dataExp: { total: 0, list: [] },
      loadExpData: false,
      templateId: "",
      tab: null,
      exptype: 0,
      dataJavelin: { total: 0, list: [] },
      loadJavelin: false,
      summary: true,
      expitems: [
        { value: 0, text: "All" },
        { value: 1, text: "Concierge" },
        { value: 2, text: "Interview" },
        { value: 3, text: "Wizard of Oz" }
      ],
      headers: [
        { text: "Date", value: "date", sortable: false, align: "start" },
        { text: "Name", value: "name", sortable: false, align: "start" },
        { text: "", value: "action", sortable: false, align: "end" }
      ],
      dialogDelete: false,
      loadDelete: false,
      deleteId: ""
    };
  },
  watch: {
    exptype: "loadSummary"
  },
  mounted() {
    this.showTemplateExp();
  },
  methods: {
    loadSummary() {
      this.summary = false;
      switch (this.exptype) {
        case 0:
          // this.summary = false;
          this.dataJavelin = { total: 0, list: [] };
          this.getConciergeData();
          this.getInterviewData();
          this.getWoZData();
          break;
        case 1:
          // this.summary = false;
          this.dataJavelin = { total: 0, list: [] };
          this.getConciergeData();
          break;
        case 2:
          // this.summary = false;
          this.dataJavelin = { total: 0, list: [] };
          this.getInterviewData();
          break;
        case 3:
          // this.summary = false;
          this.dataJavelin = { total: 0, list: [] };
          this.getWoZData();
          break;
        default:
          break;
      }
    },
    showTemplateExp: function() {
      this.showTemplate = true;
      this.axios
        .get(config.baseUri + "/team/form_template/experiment", {
          headers: auth.getAuthHeader()
        })
        .then(res => {
          this.dataTemplate = res.data.data;
        })
        .catch(res => {
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.showTemplate = false;
        });
    },
    getExpData(id) {
      this.loadExpData = true;
      this.templateId = id;
      this.axios
        .get(
          config.baseUri +
            "/team/" +
            this.$route.params.teamId +
            "/idea/" +
            this.$route.params.ideaId +
            "/customer_segment/" +
            this.$route.params.customersegmentId +
            "/persona/" +
            this.$route.params.personaId +
            "/value_proposition/" +
            this.$route.params.valuepropositionId +
            "/experiment?experiment_template_id[]=" +
            id,
          { headers: auth.getAuthHeader() }
        )
        .then(res => {
          this.dataExp = res.data.data;
        })
        .catch(res => {
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.loadExpData = false;
        });
    },
    openBsForm(id, name) {
      this.templateId = id;
      this.templateName = name;
      this.edit = false;
      this.dialogCanvas = true;
    },
    editBsForm(template, name) {
      this.templateId = "";
      this.templateName = name;
      this.templateData = template;
      this.edit = true;
      this.dialogCanvas = true;
    },
    deleteAct(id, templateId) {
      this.deleteId = id;
      this.templateId = templateId;
      this.dialogDelete = true;
    },
    deleteBS(id, templateId) {
      this.loadDelete = true;
      this.axios
        .delete(
          config.baseUri +
            "/team/" +
            this.$route.params.teamId +
            "/idea/" +
            this.$route.params.ideaId +
            "/customer_segment/" +
            this.$route.params.customersegmentId +
            "/persona/" +
            this.$route.params.personaId +
            "/value_proposition/" +
            this.$route.params.valuepropositionId +
            "/experiment/" +
            id,
          { headers: auth.getAuthHeader() }
        )
        .then(() => {
          this.refreshDelete(templateId);
        })
        .catch(res => {
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.loadDelete = false;
        });
    },
    refreshCanvas(id) {
      this.getExpData(id);
      this.dialogCanvas = false;
    },
    refreshDelete(id) {
      this.getExpData(id);
      this.dialogCanvas = false;
    }
  }
};
</script>
<style scoped>
#javeboard td {
  background: #fff !important;
  padding: 10px;
}
#javeboard th {
  width: 200px;
}
#javeboard tr {
  border-bottom: 2px solid #e4e4e4;
}
#javeboard .jbth {
  text-align: center;
  background: #efefef;
}
</style>