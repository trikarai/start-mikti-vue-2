<template>
  <v-container>
    <v-divider />
    {{field.name}}
    <!-- https://jsfiddle.net/meyubaraj/fLbe7r72/ -->
    <v-flex xs12 class="text-xs-center text-sm-center text-md-center text-lg-center">
      <!-- {{response}} -->
      <v-expand-x-transition>
        <v-progress-circular
          v-if="progressShow"
          rotate="90"
          size="150"
          width="10"
          :value="valueDeterminate"
          color="red"
        >{{ valueDeterminate }} %</v-progress-circular>
      </v-expand-x-transition>
      <v-expand-transition>
        <template v-if="!progressShow">
          <template v-if="ext == 'pdf'">
            <v-img
              src="https://www.sandhata.com/wp-content/uploads/2016/11/pdf-icon.png"
              contain
              max-height="150"
            />
            {{imageUrl}}
          </template>
          <template v-else>
            <v-img :src="imageUrl" contain max-height="150" v-if="imageUrl" />
          </template>
        </template>
      </v-expand-transition>
      <template v-if="!uploaded">
        <v-btn
          @click.prevent="uploadFile"
          v-if="imageUrl"
          :disabled="progressShow"
          small
          fab
          color="warning"
        >
          <v-icon>cloud_upload</v-icon>
        </v-btn>
      </template>
      <v-btn fab small color="green" v-if="uploaded">
        <v-icon>check</v-icon>
      </v-btn>
      <v-text-field
        label="Select Image"
        @click="pickFile"
        v-model="imageName"
        prepend-icon="attach_file"
      ></v-text-field>
      <input
        type="file"
        style="display: none"
        ref="image"
        accept="image/*, application/pdf"
        @change="onFilePicked"
      />
    </v-flex>
  </v-container>
</template>
<script>
import bus from "@/bus";
import auth from "@/config/auth";
import * as config from "@/config/app.config";
import { formMixins } from "@/mixins/formMixins";
import { uuid } from "vue-uuid";

export default {
  mixins: [formMixins],
  props: ["field", "index"],
  components: {},
  data: function() {
    return {
      fieldId: "",
      response: "",
      clearable: true,
      readonly: true,
      valueDeterminate: 0,
      value: [],
      imageName: "",
      imageUrl: "",
      imageFile: "",
      headers: {},
      progressShow: false,
      uploaded: false,
      fileInfo: { id: "", filePath: "" },
      ext: ""
    };
  },
  created() {
    this.fieldId = this.field.id;
  },
  watch: {
    fileInfo: function() {
      var params = { field_template_id: this.fieldId, value: this.value };
      bus.$emit("getValue", params, this.index);
    }
  },
  mounted() {},
  methods: {
    pickFile() {
      this.$refs.image.click();
    },
    onFilePicked(e) {
      this.progressShow = false;
      const files = e.target.files;
      if (files[0] !== undefined) {
        this.imageName = files[0].name;
        var name = this.$uuid.v4() + "_" + files[0].name;
        var str = name.replace(/\s/g, "");
        this.headers["name"] = str;
        if (this.imageName.lastIndexOf(".") <= 0) {
          return;
        }
        const fr = new FileReader();
        fr.readAsDataURL(files[0]);
        fr.addEventListener("load", () => {
          this.imageUrl = fr.result;
          this.imageFile = files[0]; // this is an image file that can be sent to server...
        });
        this.ext = files[0].name.split(".").pop();
      } else {
        this.imageName = "";
        this.imageFile = "";
        this.imageUrl = "";
      }
    },
    uploadFile: function() {
      this.headers["Content-Type"] = "image/*";
      this.headers["Authorization"] = auth.getToken();
      var app = this;
      this.progressShow = true;
      this.axios
        .post(
          config.baseUri + "/team/" + this.$route.params.teamId + "/file",
          this.imageFile,
          {
            headers: this.headers,
            onUploadProgress(e) {
              if (e.lengthComputable) {
                // console.log((e.loaded / e.total) * 100);
                app.valueDeterminate = Math.round((e.loaded / e.total) * 100);
              }
            }
          }
        )
        .then(res => {
          this.response = res.data.data;
          this.value[0] = res.data.data.id;
          this.fileInfo = res.data.data.file_path;
          this.uploaded = true;
        })
        .catch(res => {
          this.uploaded = false;
          bus.$emit("callNotif", "error", res);
        })
        .finally(() => {
          this.progressShow = false;
        });
    }
  }
};
</script>
<style scoped>
</style>
