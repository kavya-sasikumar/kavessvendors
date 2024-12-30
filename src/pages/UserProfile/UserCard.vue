<!-- eslint-disable prettier/prettier -->
<template>
  <md-card class="md-card-profile">
    <div class="md-card-avatar" @click="$refs.fileInput.click()" style="cursor: pointer;">
      <img class="img" :src="details.logo" />
      <input type="file" accept="image/*" @change="handleImageChange" ref="fileInput" class="file-input"
        style="visibility: hidden;">
    </div>

    <md-card-content>
      <h6 class="category text-gray">Store Name</h6>
      <h4 class="card-title">{{ details.store_name }}</h4>
      <p class="card-description">
        {{ details.address }}
      </p>
      <md-button class="md-round md-burgundy" @click="updateVendor">
        <span v-if="isLoading" class="loader"></span>
        <span v-else>Update Image</span>
      </md-button>
    </md-card-content>
  </md-card>
</template>

<!-- eslint-disable prettier/prettier -->
<script>
import axios from "axios";

export default {
  name: "user-card",
  props: {
    cardUserImage: {
      type: String,
      default: require("@/assets/img/faces/marc.jpg"),
    },
  },
  data() {
    return {
      url: process.env.VUE_APP_BASE_URL,
      details: [],
      isLoading: false,
    };
  },
  mounted() {
    this.checkLoggedIn();
  },
  methods: {
    checkLoggedIn() {
      if (!localStorage.getItem("token")) {
        this.$router.push({ name: "login", query: { redirect: "/user" } });
      } else {
        this.getVendorDetails();
      }
    },
    getVendorDetails() {
      this.details = JSON.parse(localStorage.getItem("vendor"));
      this.details.commission_rate = 100 - this.details.commission_rate;
      // console.log(this.details);
    },
    handleImageChange(event) {
      const file = event.target.files[0];
      if (file) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.details.logo = e.target.result;
        };
        reader.readAsDataURL(file);
      }
      // console.log(this.details)
    },
    updateVendor() {
      this.isLoading = true;
      let stringImage;
      if (typeof this.details.logo === 'string') {
        stringImage = this.details.logo;
        delete this.details.logo;
      }

      axios.put(this.url + "api/v1/vendor/" + this.details.id + "/", this.details, {
        headers: { authorization: "Token " + localStorage.getItem("token") }
      }).then(response => {
        // console.log(response.data)
        this.details.logo = stringImage;
        this.isLoading = false;
        localStorage.setItem('vendor', JSON.stringify(this.details));
      }).catch(e => {
        this.isLoading = false;
        this.$notify({
          message:
            "An Error Occurred.",
          icon: "add_alert",
          horizontalAlign: 'right',
          verticalAlign: 'top',
          type: 'danger',
        });
      });
    },
  },
};
</script>
<style></style>
