<!-- eslint-disable prettier/prettier -->
<template>
    <div class="content">
        <div class="md-layout">

            <div class="md-layout-item md-medium-size-100 md-xsmall-size-100 md-size-100">
                <md-button class="md-burgundy" style="float: right;" @click="showAddModal"><md-icon>add</md-icon> Add
                    Products</md-button>
                <md-card class="md-card-plain">

                    <md-card-header data-background-color="burgundy">
                        <h4 class="title">List of Products <b>{{ products_total }}</b></h4>
                        <p class="category">Here is a list of your products on Kavess</p>
                    </md-card-header>
                    <md-card-content>
                        <span v-if="isLoading" class="loader"></span>
                        <md-table v-model="products" style="padding: 20px; background-color: #eeeeee;" v-else>
                            <md-table-row slot="md-table-row" slot-scope="{ item }" style="background-color: #eeeeee;">
                                <md-table-cell class="avatar" md-label="" style="width: 50px; height: 60px;"><img
                                        :src="item.image" width="50" height="60"></md-table-cell>
                                <md-table-cell md-label="Title">{{ item.title }}</md-table-cell>
                                <md-table-cell md-label="Color">{{ item.color }}</md-table-cell>
                                <md-table-cell md-label="Price">{{ item.price }}</md-table-cell>
                                <md-table-cell md-label="Stock">{{ item.stock }}</md-table-cell>
                                <md-table-cell md-label=""><md-button @click="showEditModal(item)"
                                        class="md-burgundy md-just-icon md-round"><md-icon>edit</md-icon></md-button></md-table-cell>
                            </md-table-row>

                        </md-table>
                        <md-button v-if="!isLoading && nextPage" class="md-burgundy md-block" :disabled="loading"
                            @click="getMoreProducts">
                            <span v-if="loading" class="loader"></span>
                            <span v-else>Load More</span>
                        </md-button>
                    </md-card-content>
                </md-card>
            </div>
        </div>

        <modal name="edit-modal" height="auto">
            <div class="modal-content">
                <!-- Header -->
                <div class="modal-header">
                    <h4 class="modal-title">Edit Product</h4>
                    <md-button class="md-simple md-just-icon md-round modal-close-button" @click="hideEditModal()">
                        <md-icon>clear</md-icon>
                    </md-button>
                </div>

                <!-- Body -->
                <div class="modal-body">
                    <form @submit.prevent="updateProduct" class="edit-form">
                        <!-- Image Upload -->
                        <div class="image-upload">
                            <img :src="form.image" class="preview-image" alt="Product image">
                            <input type="file" accept="image/*" @change="handleImageChange" ref="fileInput"
                                class="file-input" style="visibility: hidden;">
                            <md-button class="md-raised md-burgundy" @click="$refs.fileInput.click()">
                                <md-icon>add_photo_alternate</md-icon>
                                Change Image
                            </md-button>
                        </div>

                        <!-- Title -->
                        <md-field class="md-primary">
                            <label>Product Title</label>
                            <md-input class="md-primary" v-model="form.title" required></md-input>
                        </md-field>

                        <!-- Color -->
                        <md-field>
                            <label>Color</label>
                            <md-input v-model="form.color" required></md-input>
                        </md-field>

                        <!-- Price -->
                        <md-field>
                            <label>Price</label>
                            <md-input type="number" step="0.01" v-model="form.price" required></md-input>
                            <span class="md-prefix">$</span>
                        </md-field>

                        <!-- Stock -->
                        <md-field>
                            <label>Stock</label>
                            <md-input type="number" v-model="form.stock" required></md-input>
                        </md-field>

                        <!-- Description -->
                        <md-field>
                            <label>Description</label>
                            <md-textarea v-model="form.description" required></md-textarea>
                        </md-field>

                        <!-- Availability -->
                        <md-field>
                            <md-switch v-model="form.is_available">Available for Sale</md-switch>
                        </md-field>
                    </form>
                </div>

                <!-- Footer -->
                <div class="modal-footer">
                    <md-button class="md-simple" @click="updateProduct">
                        <span v-if="loading" class="loader"></span>
                        <span v-else>Proceed</span>
                    </md-button>
                    <md-button class="md-simple md-danger" @click="hideEditModal">CLOSE</md-button>
                </div>
            </div>
        </modal>

        <modal name="add-modal" height="auto">
            <div class="modal-content">
                <!-- Header -->
                <div class="modal-header">
                    <h4 class="modal-title">Add Product</h4>
                    <md-button class="md-simple md-just-icon md-round modal-close-button" @click="hideAddModal()">
                        <md-icon>clear</md-icon>
                    </md-button>
                </div>

                <!-- Body -->
                <div class="modal-body">
                    <form @submit.prevent="addProduct" class="edit-form">
                        <!-- Image Upload -->
                        <div class="image-upload">
                            <img :src="addForm.image" class="preview-image" alt="Product image">
                            <input type="file" accept="image/*" @change="handleImageChangeAdd" ref="fileInput"
                                class="file-input" style="visibility: hidden;">
                            <md-button class="md-raised md-burgundy" @click="$refs.fileInput.click()">
                                <md-icon>add_photo_alternate</md-icon>
                                Change Image
                            </md-button>
                        </div>

                        <!-- Title -->
                        <md-field class="md-primary">
                            <label>Product Title</label>
                            <md-input class="md-primary" v-model="addForm.title" required></md-input>
                        </md-field>

                        <!-- Color -->
                        <md-field>
                            <label>Color</label>
                            <md-input v-model="addForm.color" required></md-input>
                        </md-field>

                        <!-- Price -->
                        <md-field>
                            <label>Price</label>
                            <md-input type="number" step="0.01" v-model="addForm.price" required></md-input>
                            <span class="md-prefix">$</span>
                        </md-field>

                        <!-- Stock -->
                        <md-field>
                            <label>Stock</label>
                            <md-input type="number" v-model="addForm.stock" required></md-input>
                        </md-field>

                        <!-- Description -->
                        <md-field>
                            <label>Description</label>
                            <md-textarea v-model="addForm.description" required></md-textarea>
                        </md-field>

                        <!-- Availability -->
                        <md-field>
                            <md-switch v-model="addForm.is_available">Available for Sale</md-switch>
                        </md-field>
                    </form>
                </div>

                <!-- Footer -->
                <div class="modal-footer">
                    <md-button class="md-simple" @click="addProduct">
                        <span v-if="loading" class="loader"></span>
                        <span v-else>Proceed</span>
                    </md-button>
                    <md-button class="md-simple md-danger" @click="hideAddModal">CLOSE</md-button>
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
            products: [],
            isLoading: false,
            loading: false,
            nextPage: null,
            products_total: 0,
            rowsPerPage: 3,
            editModal: false,
            form: {},
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
                this.$router.push({ name: "login", query: { redirect: "/products" } });
            } else {
                this.getProducts();
            }
        },
        showEditModal(item) {
            this.form = item;
            this.$modal.show('edit-modal');
        },
        hideEditModal() {
            this.form = {};
            this.$modal.hide('edit-modal');
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
            // console.log(this.form)
        },
        updateProduct() {
            this.loading = true;
            let stringImage;
            if (typeof this.form.image === 'string') {
                stringImage = this.form.image;
                delete this.form.image;
            }

            axios.put(this.url + "api/v1/product/" + this.form.id + "/", this.form, {
                headers: { authorization: "Token " + localStorage.getItem("token") }
            }).then(response => {
                // console.log(response.data)
                this.form.image = stringImage;
                this.$modal.hide('edit-modal');
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
        getProducts() {
            this.isLoading = true;
            axios.get(this.url + "api/v1/vendor-products/" + localStorage.getItem("userId") + "/", {
                headers: { authorization: "Token " + localStorage.getItem("token") }
            }).then(response => {
                // console.log(response.data)
                this.products = response.data.results;
                this.products_total = response.data.total;
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
        getMoreProducts() {
            this.loading = true;
            if (this.nextPage != null) {
                axios.get(this.nextPage, {
                    headers: { authorization: "Token " + localStorage.getItem("token") }
                }).then(response => {
                    // console.log(response.data)
                    this.products = [
                        ...this.products,
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
                this.products_total += 1;
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
</style>