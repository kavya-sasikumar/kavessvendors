<!-- eslint-disable prettier/prettier -->
<template>
    <div class="content">
        <div class="md-layout">

            <div class="md-layout-item md-medium-size-100 md-xsmall-size-100 md-size-100">
                <md-card class="md-card-plain">

                    <md-card-header data-background-color="burgundy">
                        <h4 class="title">List of Orders <b>{{ orders_total }}</b></h4>
                        <p class="category">Here is a list of your orders on Kavess</p>
                    </md-card-header>
                    <md-card-content>
                        <span v-if="isLoading" class="loader"></span>
                        <md-table v-model="orders" style="padding: 20px; background-color: #eeeeee;" v-else>
                            <md-table-row slot="md-table-row" slot-scope="{ item }" style="background-color: #eeeeee;">
                                <md-table-cell class="avatar" md-label="" style="width: 50px; height: 60px;"><img
                                        :src="`${url.replace(/\/$/, '')}${item.product_details.image}`" width="50"
                                        height="60"></md-table-cell>
                                <md-table-cell md-label="Title">{{ item.product_details.title }}</md-table-cell>
                                <md-table-cell md-label="Status">{{ item.status_type }}</md-table-cell>
                                <md-table-cell md-label="Unit Price">{{ item.product_details.price }}</md-table-cell>
                                <md-table-cell md-label="Price">{{ item.price }}</md-table-cell>
                                <md-table-cell md-label="Quantity">{{ item.quantity }}</md-table-cell>
                                <md-table-cell md-label=""><md-button @click="showEditModal(item)"
                                        class="md-burgundy md-just-icon md-round"><md-icon>preview</md-icon></md-button></md-table-cell>
                            </md-table-row>

                        </md-table>
                        <md-button v-if="!isLoading && nextPage" class="md-burgundy md-block" :disabled="loading"
                            @click="getMoreOrders">
                            <span v-if="loading" class="loader"></span>
                            <span v-else>Load More</span>
                        </md-button>
                    </md-card-content>
                </md-card>
            </div>
        </div>

        <modal name="view-cust-modal" height="auto">
            <div class="modal-content">
                <!-- Header -->
                <div class="modal-header">
                    <h4 class="modal-title">Customer Details</h4>
                    <md-button class="md-simple md-just-icon md-round modal-close-button" @click="hideEditModal()">
                        <md-icon>clear</md-icon>
                    </md-button>
                </div>

                <!-- Body -->
                <div class="modal-body">
                    <form @submit.prevent="updateProduct" class="edit-form">

                        <!-- First Name -->
                        <md-field class="md-primary">
                            <label>First Name</label>
                            <md-input :disabled="true" class="md-primary" v-model="form.buyer.first_name"></md-input>
                        </md-field>

                        <!-- Last Name -->
                        <md-field>
                            <label>Last Name</label>
                            <md-input :disabled="true" v-model="form.buyer.last_name"></md-input>
                        </md-field>

                        <!-- Email -->
                        <md-field>
                            <label>Email</label>
                            <md-input :disabled="true" v-model="form.buyer.email"></md-input>
                        </md-field>

                        <!-- Address -->
                        <md-field>
                            <label>Address</label>
                            <md-textarea :disabled="true" v-model="form.address"></md-textarea>
                        </md-field>

                        <!-- Status -->
                        <md-field>
                            <label>Shipping Status</label>
                            <md-select v-model="form.status_type">
                                <md-option value="PENDING">Pending</md-option>
                                <md-option value="SHIPPED">Shipped</md-option>
                                <md-option value="DELIVERED">Delivered</md-option>
                                <md-option value="CANCELLED">Cancelled</md-option>
                            </md-select>
                        </md-field>
                    </form>
                </div>

                <!-- Footer -->
                <div class="modal-footer">
                    <md-button class="md-simple" @click="updateOrder">
                        <span v-if="loading" class="loader"></span>
                        <span v-else>Proceed</span>
                    </md-button>
                    <md-button class="md-simple md-danger" @click="hideEditModal">CLOSE</md-button>
                </div>
            </div>
        </modal>

    </div>
</template>

<!-- eslint-disable prettier/prettier -->
<script>
import axios from "axios";
// import { Modal } from '@/components';

export default {
    components: {
    },
    data() {
        return {
            url: process.env.VUE_APP_BASE_URL,
            orders: [],
            isLoading: false,
            loading: false,
            nextPage: null,
            orders_total: 0,
            rowsPerPage: 3,
            editModal: false,
            form: {
                buyer: {
                    first_name: '',
                    last_name: '',
                    email: '',
                }
            },
            addForm: {
                image: '',
                title: '',
                color: '',
                price: '',
                stock: '',
                description: '',
                is_available: ''
            },
        };
    },
    mounted() {
        this.checkLoggedIn();
    },
    methods: {
        checkLoggedIn() {
            if (!localStorage.getItem("token")) {
                this.$router.push({ name: "login", query: { redirect: "/orders" } });
            } else {
                this.getOrders();
            }
        },
        showEditModal(item) {
            this.form = item;
            this.$modal.show('view-cust-modal');
        },
        hideEditModal() {
            this.form = {};
            this.$modal.hide('view-cust-modal');
        },
        handleImageChange(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.form.image = e.target.result;
                };
                reader.readAsDataURL(file);
            }
            console.log(this.form)
        },
        updateOrder() {
            this.loading = true;

            axios.put(this.url + "api/v1/orderitem/" + this.form.id + "/", this.form, {
                headers: { authorization: "Token " + localStorage.getItem("token") }
            }).then(response => {
                // console.log(response.data)
                this.$modal.hide('view-cust-modal');
                this.form = {};
                this.loading = false;
            }).catch(e => {
                this.loading = false;
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
        getOrders() {
            this.isLoading = true;
            let vendor_data = JSON.parse(localStorage.getItem("vendor"))
            axios.get(this.url + "api/v1/orderitems/" + vendor_data.id + "/", {
                headers: { authorization: "Token " + localStorage.getItem("token") }
            }).then(response => {
                console.log(response.data)
                this.orders = response.data.results;
                this.orders_total = response.data.total;
                this.nextPage = response.data.links.next;
                this.isLoading = false;
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
        getMoreOrders() {
            this.loading = true;
            if (this.nextPage != null) {
                axios.get(this.nextPage, {
                    headers: { authorization: "Token " + localStorage.getItem("token") }
                }).then(response => {
                    // console.log(response.data)
                    this.orders = [
                        ...this.orders,
                        ...response.data.results,
                    ];
                    this.nextPage = response.data.links.next;
                    this.loading = false;
                }).catch(e => {
                    this.loading = false;
                    this.$notify({
                        message:
                            "An Error Occurred.",
                        icon: "add_alert",
                        horizontalAlign: 'right',
                        verticalAlign: 'top',
                        type: 'danger',
                    });
                });
            }
        },
        showAddModal() {
            // this.addForm = this.form;
            this.$modal.show('add-modal');
        },
        hideAddModal() {
            this.$modal.hide('add-modal');
        },
        handleImageChangeAdd(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    this.addForm.image = e.target.result;
                    this.addForm.imageUpload = file;
                };
                reader.readAsDataURL(file);
            }
        },
        addProduct() {
            this.loading = true;

            let formData = new FormData();
            formData.append("image", this.addForm.imageUpload);
            formData.append("title", this.addForm.title);
            formData.append("color", this.addForm.color);
            formData.append("price", this.addForm.price);
            formData.append("stock", this.addForm.stock);
            formData.append("description", this.addForm.description);
            formData.append("is_available", this.addForm.is_available);
            formData.append("category", '1');
            let vendor_data = JSON.parse(localStorage.getItem("vendor"))
            formData.append("user", vendor_data.id);

            axios.post(this.url + "api/v1/create-product/", formData, {
                headers: { authorization: "Token " + localStorage.getItem("token") }
            }).then(response => {

                const newProduct = response.data;
                this.products = [
                    newProduct,
                    ...this.products,
                ];
                this.orders_total += 1;
                this.$modal.hide('add-modal');
                this.loading = false;
            }).catch(e => {
                // console.log(e)
                this.loading = false;
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
    }
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

.modal-content {
    background: white;
    border-radius: 8px;
    overflow: hidden;
}

.modal-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding: 24px;
    border-bottom: 1px solid rgba(0, 0, 0, 0.1);
}

.modal-title {
    font-size: 24px;
    font-weight: 400;
    margin: 0;
    color: rgba(0, 0, 0, 0.87);
}

.modal-close-button {
    min-width: 40px !important;
    width: 40px !important;
    height: 40px !important;
    margin: 0 !important;
}

.modal-close-button .md-icon {
    font-size: 20px !important;
    color: rgba(0, 0, 0, 0.54);
}

.modal-body {
    padding: 24px;
    max-height: 70vh;
    overflow-y: auto;
}

.edit-form {
    display: flex;
    flex-direction: column;
    gap: 20px;
}

.image-upload {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 12px;
    padding: 16px;
    border: 1px dashed rgba(0, 0, 0, 0.2);
    border-radius: 4px;
}

.preview-image {
    width: 200px;
    height: 200px;
    object-fit: cover;
    border-radius: 4px;
}

.file-input {
    display: none;
}

.modal-footer {
    /* padding: 16px 24px; */
    display: flex;
    justify-content: flex-end;
    gap: 8px;
    border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.md-button.md-simple {
    margin: 0;
    padding: 0 16px;
    min-width: 90px;
    font-weight: 500;
}

.md-button.md-danger {
    color: #f44336 !important;
}

@media (max-width: 600px) {
    .modal-header {
        padding: 16px;
    }

    .modal-title {
        font-size: 20px;
    }

    .modal-body {
        padding: 16px;
    }

    .modal-footer {
        padding: 12px 16px;
    }
}

.modal-content {
    overflow: visible !important;
    position: relative;
}

.md-select-menu {
    z-index: 1050 !important;
    position: absolute;
}
</style>