<template>
    <v-text-field
        prepend-inner-icon="docs"
        :model-value="modelValue"
        @update:model-value="$emit('update:modelValue', value)"
    >
        <template v-slot:append>
            <input
                :accept="accept"
                :ref="uniqueId"
                class="position-absolute"
                type="file"
                style="height: 0; width: 0; top: 0"
                @change="onFileChange"
            />

            <v-icon v-if="!modelValue && uploadable" @click="openUploadDialog">
                arrow_upload_progress
            </v-icon>

            <v-dialog
                v-else-if="modelValue && extension === 'pdf'"
                transition="dialog-bottom-transition"
                v-model="dialog"
                fullscreen
            >
                <template v-slot:activator="{ props: activatorProps }">
                    <v-icon v-bind="activatorProps">folder_open</v-icon>
                </template>

                <template v-slot:default="{ isActive }">
                    <widget-pdf
                        :state="isActive.value"
                        :src="`${backendUrl}?path=${modelValue}`"
                        @close="dialog = false"
                    ></widget-pdf>
                </template>
            </v-dialog>

            <v-icon
                v-else-if="modelValue && extension !== 'pdf'"
                @click="downloadFileUpload"
            >
                downloading
            </v-icon>
        </template>

        <template v-slot:append-inner v-if="modelValue && deletable">
            <v-dialog max-width="340">
                <template v-slot:activator="{ props: activatorProps }">
                    <v-icon color="deep-orange" v-bind="activatorProps"
                        >delete</v-icon
                    >
                </template>

                <template v-slot:default="{ isActive }">
                    <v-card
                        text="Apakah Anda akan menghapus file ini secara permanent dari server?"
                        title="Hapus File"
                    >
                        <template v-slot:actions>
                            <v-btn
                                class="ml-auto"
                                color="grey-darken-1"
                                text="Batal"
                                @click="isActive.value = false"
                            ></v-btn>

                            <v-btn
                                color="deep-orange"
                                text="Hapus"
                                @click="deleteFileUpload"
                            ></v-btn>
                        </template>
                    </v-card>
                </template>
            </v-dialog>
        </template>
    </v-text-field>
</template>

<script>
export default {
    name: "file-upload",

    props: {
        accept: {
            type: String,
            default: ".pdf,application/pdf",
        },

        backendUrl: {
            type: String,
            default: "",
        },

        callback: {
            type: Function,
            default: () => {},
        },

        deletable: {
            type: Boolean,
            default: false,
        },

        extension: {
            type: String,
            default: "pdf",
        },

        uploadable: {
            type: Boolean,
            default: false,
        },

        slug: {
            type: String,
            default: null,
        },

        path: {
            type: String,
            default: null,
        },

        modelValue: String,
    },

    data: () => ({
        dialog: false,
        fileinfo: null,
        uniqueId: Math.floor(1000 + Math.random() * 9000),
    }),

    methods: {
        openUploadDialog() {
            this.$refs[this.uniqueId].click();
        },

        downloadFileUpload() {
            console.log("zz");
        },

        deleteFileUpload() {
            this.$http(`${this.backendUrl}`, {
                method: "DELETE",
                params: {
                    path: this.modelValue,
                },
            }).then(() => {
                this.$emit("update:modelValue", null);
            });
        },

        digest: async function (message) {
            return Array.prototype.map
                .call(
                    new Uint8Array(
                        await crypto.subtle.digest(
                            "SHA-256",
                            new TextEncoder().encode(message)
                        )
                    ),
                    (x) => ("0" + x.toString(16)).slice(-2)
                )
                .join("");
        },

        fileUUID: function (file) {
            return new Promise((resolve, reject) => {
                try {
                    let reader = new FileReader();

                    reader.onload = (event) => {
                        this.digest(event.target.result).then((uuid) => {
                            resolve(uuid);
                        });
                    };

                    reader.readAsText(file);
                } catch (error) {
                    reject(error);
                }
            });
        },

        onFileChange(event) {
            let file = event.target.files.item(0);

            this.fileUUID(file).then((uuid) => {
                this.postUploadFile({
                    file: file,
                    extension: this.extension,
                    size: this.size,
                    uuid: uuid,
                    slug: this.slug,
                });
            });
        },

        onFileDelete() {
            this.$emit("input", null);
            this.$refs[this.uniqueId].value = null;
        },

        postUploadFile(data) {
            this.$http(`${this.backendUrl}`, {
                method: "POST",
                contentType: "multipart/form-data",
                params: data,
            })
                .then((response) => {
                    this.callback(response);
                })
                .catch((response) => {
                    console.log(response);
                });
        },

        updateValue(value) {
            if (value.length > 0) {
                this.$emit("input", value);
            } else {
                this.$emit("input", null);
            }
        },
    },
};
</script>
