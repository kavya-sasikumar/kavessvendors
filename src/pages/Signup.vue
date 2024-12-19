<!-- eslint-disable prettier/prettier -->
<template>
    <div class="signup-page">
        <notifications></notifications>
        <div class="signup-container">
            <h1 class="signup-title">Register</h1>
            <form @submit.prevent="handleSignup" class="signup-form">
                <div class="form-group">
                    <label for="firstName" class="form-label">Store Name</label>
                    <input type="text" id="firstName" v-model="credentials.store_name" class="form-input" required />
                </div>
                <div class="form-group">
                    <label for="email" class="form-label">Email</label>
                    <input type="email" id="email" v-model="credentials.email" class="form-input" required />
                </div>

                <div class="form-group">
                    <label for="firstName" class="form-label">Username</label>
                    <input type="text" id="firstName" v-model="credentials.username" class="form-input" required />
                </div>
                <div class="form-group">
                    <label for="phone" class="form-label">Phone Number</label>
                    <input type="tel" id="phone" v-model="credentials.phone_number" class="form-input" required>
                </div>


                <div class="form-group">
                    <label for="password" class="form-label">Password</label>
                    <input type="password" id="password" v-model="credentials.password1" class="form-input" required />
                </div>
                <div class="form-group">
                    <label class="form-checkbox">
                        <input type="checkbox" style="accent-color: #7d2248 !important;" v-model="agreedToTerms"
                            required />
                        I agree to the
                        <a href="https://kavess.com/terms" target="__blank" class="terms-link">Terms and Conditions</a>
                    </label>
                </div>
                <button type="submit" class="signup-btn" :disabled="loading">
                    <span v-if="loading" class="loader"></span>
                    <span v-else>Get Started</span>
                </button>
                <div class="form-group" style="margin-top: 10px;">
                    <label class="form-checkbox">
                        Already have an account?
                        <router-link to="login" class="terms-link">Login</router-link>
                    </label>
                </div>
            </form>
            <!-- <div class="signup-footer">
                <p>or be classical</p>
                <div class="social-icons">
                    <a href="#" class="social-icon"><i class="fab fa-twitter"></i></a>
                    <a href="#" class="social-icon"><i class="fab fa-basketball-ball"></i></a>
                    <a href="#" class="social-icon"><i class="fab fa-facebook-f"></i></a>
                </div>
            </div> -->
        </div>
    </div>
</template>
<!-- eslint-disable prettier/prettier -->
<script>
import axios from "axios";
export default {

    data() {
        return {
            url: process.env.VUE_APP_BASE_URL,
            credentials: {},
            loading: false,
            firstName: "",
            email: "",
            password: "",
            agreedToTerms: false,
        };
    },
    mounted() {
        this.checkLoggedIn();
    },
    methods: {
        checkLoggedIn() {
            if (localStorage.getItem("token")) {
                this.$router.push("/");
            }
        },
        handleSignup() {
            if (this.agreedToTerms && this.credentials.store_name && this.credentials.email && this.credentials.username && this.credentials.phone_number && this.credentials.password1) {

                this.loading = true;
                this.credentials.password2 = this.credentials.password1;
                this.credentials.username = this.credentials.username.toLowerCase();

                axios.post(this.url + 'auth/registration/', this.credentials).then((response) => {
                    console.log(response.data)
                    this.credentials.password = this.credentials.password1;
                    axios.post(this.url + 'auth/login/', this.credentials).then((response) => {
                        // console.log(response.data)
                        localStorage.setItem("token", response.data.key);
                        axios.get(this.url + "api/v1/check-user/", {
                            headers: { authorization: "Token " + localStorage.getItem("token") }
                        }).then(response2 => {
                            localStorage.setItem("userId", response2.data.results[0].user_data);
                            localStorage.setItem("username", response2.data.results[0].username);
                            this.credentials.address = 'Somewhere in the world';
                            this.credentials.user = response2.data.results[0].user_data;
                            axios.post(this.url + "api/v1/create-vendor/", this.credentials, {
                                headers: { authorization: "Token " + localStorage.getItem("token") }
                            }).then(response4 => {
                                axios.get(this.url + "api/v1/vendor/" + response2.data.results[0].user_data + "/", {
                                    headers: { authorization: "Token " + localStorage.getItem("token") }
                                }).then(response3 => {
                                    localStorage.setItem("vendor", JSON.stringify(response3.data));
                                    this.$router.push("/");
                                    this.loading = false;
                                });
                            });
                        });
                    });
                }).catch(e => {
                    this.$notify({
                        message:
                            "An Error Occurred.",
                        icon: "add_alert",
                        horizontalAlign: 'right',
                        verticalAlign: 'top',
                        type: 'danger',
                    });
                })

            } else {
                this.loading = false;
                this.$notify({
                    message:
                        "Kindly fill all fields",
                    icon: "add_alert",
                    horizontalAlign: 'right',
                    verticalAlign: 'top',
                    type: 'danger',
                });
            }
        },
    },
};
</script>
<!-- eslint-disable prettier/prettier -->
<style>
@font-face {
    font-family: avenir-medium;
    src: url('@/assets/fonts/AvenirLTStd-Medium.otf');
}

@font-face {
    font-family: avenir-light;
    src: url('@/assets/fonts/AvenirLTStd-Light.otf');
}

.signup-page {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
    background-color: #fff;
    color: #fff;
}

.signup-container {
    background-color: #7d2248;
    border-radius: 8px;
    padding: 40px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
    max-width: 400px;
    width: 100%;
}

.signup-title {
    font-family: avenir-medium;
    font-size: 24px;
    font-weight: 600;
    margin-bottom: 24px;
    color: #fff;
}

.signup-form .form-group {
    margin-bottom: 16px;
}

.form-label {
    display: block;
    font-family: avenir-light !important;
    font-size: 14px;
    font-weight: 500;
    margin-bottom: 4px;
    color: #fff;
}

.form-input {
    width: 100%;
    padding: 12px 16px;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 16px;
}

.form-checkbox {
    display: flex;
    align-items: center;
    font-size: 14px;
}

.form-checkbox input[type="checkbox"] {
    margin-right: 8px;
}

.terms-link {
    color: #fff !important;
    text-decoration: underline !important;
}

.signup-btn {
    width: 100%;
    background-color: #fff;
    color: #7d2248;
    border: none;
    border-radius: 4px;
    padding: 12px 16px;
    font-size: 16px;
    cursor: pointer;
}

.signup-footer {
    margin-top: 24px;
    text-align: center;
}

.social-icons {
    display: flex;
    justify-content: center;
    margin-top: 16px;
}

.social-icon {
    display: inline-flex;
    justify-content: center;
    align-items: center;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    background-color: #7d2248;
    color: #fff;
    margin: 0 8px;
    text-decoration: none;
}

.social-icon i {
    font-size: 16px;
}

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
