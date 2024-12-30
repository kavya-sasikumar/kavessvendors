<!-- eslint-disable prettier/prettier -->
<template>
  <form>
    <md-card>
      <md-card-header :data-background-color="dataBackgroundColor">
        <h4 class="title">Edit Profile</h4>
        <p class="category">Complete your profile</p>
      </md-card-header>

      <md-card-content>
        <div class="md-layout">
          <div class="md-layout-item md-small-size-100 md-size-33">
            <md-field>
              <label>Store Name</label>
              <md-input v-model="details.store_name"></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-small-size-100 md-size-33">
            <md-field>
              <label>Phone Number</label>
              <md-input v-model="details.phone_number" type="text"></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-small-size-100 md-size-33">
            <md-field>
              <label>Rating</label>
              <md-input v-model="details.rating" type="text" disabled></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-small-size-100 md-size-50">
            <md-field>
              <label>Account Balance</label>
              <md-input v-model="details.account_balance" type="text" disabled></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-small-size-100 md-size-50">
            <md-field>
              <label>Commission Rate (%)</label>
              <md-input v-model="details.commission_rate" type="text" disabled></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-small-size-100 md-size-100">
            <md-field>
              <label>Bank Account Details</label>
              <md-input v-model="details.bank_account_details" type="text"></md-input>
            </md-field>
          </div>
          <div class="md-layout-item md-size-100">
            <md-field maxlength="5">
              <label>Address</label>
              <md-textarea v-model="details.address"></md-textarea>
            </md-field>
          </div>
          <div class="md-layout-item md-size-100 text-right">
            <md-button class="md-raised md-burgundy" @click="updateVendor()">
              <span v-if="isLoading" class="loader"></span>
              <span v-else>Update Profile</span>
            </md-button>
          </div>
        </div>
      </md-card-content>
    </md-card>
  </form>
</template>

<!-- eslint-disable prettier/prettier -->
<script>
import axios from "axios";

export default {
  name: "edit-profile-form",
  props: {
    dataBackgroundColor: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      url: process.env.VUE_APP_BASE_URL,
      details: [],
      isLoading: false,
      username: null,
      disabled: null,
      emailadress: null,
      lastname: null,
      firstname: null,
      address: null,
      city: null,
      country: null,
      code: null,
      aboutme:
        "Lamborghini Mercy, Your chick she so thirsty, I'm in that two seat Lambo.",
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
        this.$notify({
          message:
            "Successfully Updated Profile",
          icon: "add_alert",
          horizontalAlign: 'right',
          verticalAlign: 'top',
          type: 'success',
        });
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

<!-- eslint-disable prettier/prettier -->
<style>
.loader {
  border: 4px solid #f3f3f3;
  /* Light grey */
  border-top: 4px solid #7d2248;
  /* Blue */
  border-radius: 50%;
  width: 24px;
  height: 24px;
  animation: spin 1s linear infinite;
  display: inline-block;
  vertical-align: middle;
  margin-right: 5px;
}

@keyframes spin {
  0% {
    transform: rotate(0deg);
  }

  100% {
    transform: rotate(360deg);
  }
}
</style>
