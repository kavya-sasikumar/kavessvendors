<!-- eslint-disable prettier/prettier -->
<!-- Modal.vue -->
<template>
    <transition name="modal">
        <div class="modal-mask" v-if="show" @click.self="$emit('close')">
            <div class="modal-container md-elevation-7">
                <div class="modal-header">
                    <slot name="header">
                        <h3>Default Header</h3>
                    </slot>
                </div>

                <div class="modal-body">
                    <slot name="body">
                        Default body content
                    </slot>
                </div>

                <div class="modal-footer">
                    <slot name="footer">
                        <md-button class="md-simple" @click="$emit('close')">Close</md-button>
                    </slot>
                </div>
            </div>
        </div>
    </transition>
</template>

<!-- eslint-disable prettier/prettier -->
<script>
export default {
    name: 'Modal',
    props: {
        show: {
            type: Boolean,
            default: false
        }
    },
    watch: {
        show(newVal) {
            if (newVal) {
                document.body.style.overflow = 'hidden';
            } else {
                document.body.style.overflow = 'auto';
            }
        }
    },
    mounted() {
        document.addEventListener('keydown', this.handleEscape);
    },
    beforeDestroy() {
        document.removeEventListener('keydown', this.handleEscape);
    },
    methods: {
        handleEscape(e) {
            if (e.key === 'Escape' && this.show) {
                this.$emit('close');
            }
        }
    }
}
</script>

<!-- eslint-disable prettier/prettier -->
<style scoped>
.modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    transition: opacity 0.3s ease;
}

.modal-container {
    width: 90%;
    max-width: 500px;
    background-color: #fff;
    border-radius: 4px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.33);
    transition: all 0.3s ease;
}

.modal-header {
    padding: 20px;
    border-bottom: 1px solid #eee;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.modal-body {
    padding: 20px;
    max-height: 70vh;
    overflow-y: auto;
}

.modal-footer {
    padding: 20px;
    border-top: 1px solid #eee;
    display: flex;
    justify-content: flex-end;
    gap: 10px;
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
    transition: opacity 0.3s;
}

.modal-enter,
.modal-leave-to {
    opacity: 0;
}

.modal-enter .modal-container,
.modal-leave-to .modal-container {
    transform: scale(0.9);
}
</style>