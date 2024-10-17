<template>
  <v-container>
    <v-row class="d-flex justify-center mt-5 pt-5" style="min-height: 100vh">
      <v-col cols="12" md="6" lg="5">
        <v-card title="Upload Area">
          <v-card-text class="d-flex flex-column gap-4">
            <v-text-field
              v-model="form.endpoint"
              label="Endpoint"
              placeholder="http[s]://[yourSite]/[yourEndPoint]"
              required
            ></v-text-field>

            <v-text-field
              v-model="form.parameter_name"
              label="Name of Parameter"
              placeholder="Specific name for parameter required by your API endpoint"
              required
            ></v-text-field>

            <!-- Method Select -->
            <v-select
              v-model="form.method"
              :items="methods"
              label="Method"
              placeholder="Select HTTP Method"
              required
            ></v-select>

            <!-- Filetype Select -->
            <v-select
              v-model="form.filetype"
              :items="filetypes"
              label="Filetype"
              placeholder="Select Filetype"
              required
            ></v-select>

            <!-- Send As Select -->
            <v-select
              v-model="form.send_as"
              :items="send_as_options"
              label="Send As"
              placeholder="Send As Base64 or File"
              required
            ></v-select>

            <!-- Auth Token Checkbox -->
            <v-checkbox
              v-model="form.use_auth_token"
              label="Use Auth Token"
              hide-details
            ></v-checkbox>

            <v-text-field
              v-if="form.use_auth_token"
              v-model="form.auth_token"
              label="Auth token"
              placeholder="Bearer [your token]"
              required
            ></v-text-field>

            <!-- Add Parameters Button -->
            <v-btn color="primary" @click="showParamModal = true">
              Tambah Parameter
            </v-btn>

            <!-- Modal for adding parameters -->
            <v-dialog v-model="showParamModal" max-width="500">
              <v-card>
                <v-card-title class="headline">Tambah Parameter</v-card-title>
                <v-card-text>
                  <v-container>
                    <v-row
                      v-for="(param, index) in form.additionalParams"
                      :key="index"
                    >
                      <v-col cols="5">
                        <v-text-field
                          v-model="param.key"
                          label="Key"
                          placeholder="Enter key"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="5">
                        <v-text-field
                          v-model="param.value"
                          label="Value"
                          placeholder="Enter value"
                          required
                        ></v-text-field>
                      </v-col>
                      <v-col cols="2">
                        <v-btn icon @click="removeParam(index)">
                          <v-icon>mdi-delete</v-icon>
                        </v-btn>
                      </v-col>
                    </v-row>
                    <v-btn color="primary" @click="addParam">
                      <v-icon left>mdi-plus</v-icon> Tambah Parameter
                    </v-btn>
                  </v-container>
                </v-card-text>
                <v-card-actions>
                  <v-spacer></v-spacer>
                  <v-btn
                    color="green darken-1"
                    text
                    @click="showParamModal = false"
                    >Tutup</v-btn
                  >
                </v-card-actions>
              </v-card>
            </v-dialog>

            <!-- Avatar and Upload -->
            <div v-if="form.filetype === 'image'" class="d-flex gap-4">
              <!-- 👉 Avatar -->
              <v-avatar
                rounded="lg"
                size="100"
                class="me-6"
                :src="data.profile_picture"
              />

              <!-- 👉 Upload Photo -->
              <div class="d-flex flex-column justify-center gap-2">
                <div class="d-flex flex-wrap gap-2">
                  <v-btn color="primary" @click="triggerFileUpload">
                    <v-icon icon="mdi-cloud-upload" class="d-sm-none" />
                    <span class="d-none d-sm-block">Send your picture</span>
                  </v-btn>

                  <input
                    ref="refInputEl"
                    type="file"
                    :accept="fileAccept"
                    hidden
                    @input="sendFile"
                  />
                </div>

                <p class="mb-0 gray-text">Only jpg, jpeg, png.</p>
              </div>
            </div>

            <div v-if="form.filetype === 'excel'" class="d-flex gap-4">
              <v-btn color="primary" @click="triggerFileUpload">
                <v-icon icon="mdi-cloud-upload" class="d-sm-none" />
                <span class="d-none d-sm-block">Upload Excel</span>
              </v-btn>

              <input
                ref="refInputEl"
                type="file"
                :accept="fileAccept"
                hidden
                @input="sendFile"
              />
            </div>
          </v-card-text>
        </v-card>
      </v-col>

      <v-col cols="12" md="6" lg="5">
        <!-- Log Area -->
        <v-card title="Error Log Area" color="red" dark>
          <v-card-text v-if="logMessage">
            {{ logMessageStatus }}<br /><br />{{ logMessage }}
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import {
  VRow,
  VCol,
  VCard,
  VCardText,
  VAvatar,
  VBtn,
  VIcon,
  VTextField,
  VSelect,
  VCheckbox,
  VDialog,
} from "vuetify/components";
import axios from "axios";
import Swal from "sweetalert2";

export default {
  name: "UploadAvatar",
  components: {
    VRow,
    VCol,
    VCard,
    VCardText,
    VAvatar,
    VBtn,
    VIcon,
    VTextField,
    VSelect,
    VCheckbox,
    VDialog,
  },
  data() {
    return {
      data: {
        profile_picture: "",
      },
      form: {
        endpoint: "http://localhost:8000/api",
        parameter_name: "file",
        use_auth_token: true,
        auth_token: "Bearer ",
        method: "POST",
        filetype: "image",
        send_as: "base64",
        additionalParams: [], // For additional key-value parameters
      },
      methods: ["POST", "PUT"],
      filetypes: ["image", "excel"],
      send_as_options: ["base64", "file"],
      showParamModal: false, // Modal visibility state
      refInputEl: null,
      logMessage: "",
      logMessageStatus: "",
    };
  },
  computed: {
    fileAccept() {
      return this.form.filetype === "image" ? ".jpeg,.png,.jpg" : ".xlsx,.xls";
    },
  },
  methods: {
    triggerFileUpload() {
      this.$refs.refInputEl.click();
    },
    async sendFile(event) {
      const file = event.target.files[0];
      if (file) {
        if (this.form.send_as === "base64") {
          await this.sendAsBase64(file);
        } else {
          await this.sendAsFile(file);
        }
      }
    },
    async sendAsBase64(file) {
      const fileReader = new FileReader();
      fileReader.readAsDataURL(file);
      fileReader.onload = async () => {
        try {
          this.data.profile_picture = fileReader.result;

          const formData = new FormData();
          formData.append(this.form.parameter_name, fileReader.result);

          this.appendAdditionalParams(formData); // Append additional params

          const response = await this.sendRequest(formData);

          if (response.data.success) {
            this.showSuccess(response.data.success.message);
          }
        } catch (error) {
          this.handleError(error);
        }
      };
    },
    async sendAsFile(file) {
      const formData = new FormData();
      formData.append(this.form.parameter_name, file);

      this.appendAdditionalParams(formData); // Append additional params

      try {
        const response = await this.sendRequest(formData);

        if (response.data.success) {
          this.showSuccess(response.data.success.message);
        }
      } catch (error) {
        this.handleError(error);
      }
    },
    async sendRequest(data) {
      const headers = this.form.use_auth_token
        ? { Authorization: `${this.form.auth_token}` }
        : {};

      return await axios({
        method: this.form.method.toLowerCase(),
        url: this.form.endpoint,
        data: data,
        headers: headers,
      });
    },
    appendAdditionalParams(formData) {
      // Append additional key-value params to the FormData
      this.form.additionalParams.forEach((param) => {
        formData.append(param.key, param.value);
      });
    },
    addParam() {
      this.form.additionalParams.push({ key: "", value: "" });
    },
    removeParam(index) {
      this.form.additionalParams.splice(index, 1);
    },
    showSuccess(message) {
      Swal.fire({
        toast: true,
        position: "top",
        iconColor: "white",
        color: "white",
        background: "green",
        showConfirmButton: false,
        timerProgressBar: true,
        timer: 1500,
        icon: "success",
        title: message,
      });
    },
    handleError(error) {
      console.log(error);

      Swal.fire({
        toast: true,
        position: "top",
        iconColor: "white",
        color: "white",
        background: "red",
        showConfirmButton: false,
        timerProgressBar: true,
        timer: 1500,
        icon: "error",
        title: "Terjadi kesalahan saat upload file!",
      });

      if (error.response) {
        this.logMessageStatus = "Error status: " + error.response.status;
        this.logMessage =
          "Response from server: " + JSON.stringify(error.response.data);
      } else {
        this.logMessageStatus = "Error status: Unknown";
        this.logMessage = "Error message: " + error.message;
      }
    },
  },
};
</script>

<style scoped>
.d-flex {
  display: flex;
}
.d-sm-none {
  display: none;
}
.d-none {
  display: none;
}
.d-sm-block {
  display: block;
}
.me-6 {
  margin-right: 1.5rem;
}
.gray-text {
  color: gray;
}
</style>
