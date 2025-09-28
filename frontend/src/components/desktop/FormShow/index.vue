<template>
    <v-app-bar
        :color="`${theme}-lighten-5`"
        :order="1"
        height="72"
        scroll-behavior="elevate"
        scroll-threshold="87"
    >
        <v-btn
            icon
            @click="
                navbackTo ? $router.push({ name: navbackTo }) : openFormData()
            "
        >
            <v-icon>arrow_back</v-icon>
        </v-btn>

        <v-toolbar-title class="text-body-2 font-weight-bold text-uppercase">{{
            page.name
        }}</v-toolbar-title>

        <v-spacer></v-spacer>

        <template v-if="softdelete">
            <v-btn icon v-if="canRestore">
                <v-icon>restore</v-icon>

                <form-confirm icon="restore" title="pulihkan">
                    Proses ini akan memulihkan data ini dari status trashed.

                    <template v-slot:actions="{ isActive }">
                        <v-spacer></v-spacer>

                        <v-btn
                            :color="`${theme}-lighten-2`"
                            class="text-white mr-2"
                            variant="flat"
                            @click="isActive.value = false"
                            >batal</v-btn
                        >

                        <v-btn
                            color="deep-orange"
                            text="Pulihkan"
                            @click="
                                postFormRestore(() => (isActive.value = false))
                            "
                        ></v-btn>
                    </template>
                </form-confirm>
            </v-btn>

            <v-btn color="orange" icon v-if="canDestroy">
                <v-icon>delete_forever</v-icon>

                <form-confirm icon="delete_forever" title="Hapus Permanen?">
                    Proses ini akan menghapus data secara permanen, proses ini
                    tidak dapat di pulihkan setelah di lakukan.

                    <template v-slot:actions="{ isActive }">
                        <v-spacer></v-spacer>

                        <v-btn
                            :color="`${theme}-lighten-2`"
                            class="text-white mr-2"
                            variant="flat"
                            @click="isActive.value = false"
                            >batal</v-btn
                        >

                        <v-btn
                            color="deep-orange"
                            text="Hapus"
                            @click="
                                postFormForceDelete(
                                    () => (isActive.value = false)
                                )
                            "
                        ></v-btn>
                    </template>
                </form-confirm>
            </v-btn>
        </template>

        <template v-else>
            <v-btn v-if="!hideEdit && canEdit" icon @click="openFormEdit">
                <v-icon>edit</v-icon>

                <v-tooltip activator="parent" location="bottom">Ubah</v-tooltip>
            </v-btn>

            <v-btn v-if="!hideDelete && canDelete" color="deep-orange" icon>
                <v-icon>delete</v-icon>

                <form-confirm icon="delete" title="Hapus data ini?">
                    Proses ini akan juga menghapus semua data yang terkait pada
                    data ini.

                    <template v-slot:actions="{ isActive }">
                        <v-row dense>
                            <v-col cols="6">
                                <v-btn
                                    :color="theme"
                                    rounded="pill"
                                    variant="outlined"
                                    block
                                    @click="isActive.value = false"
                                    >BATAL</v-btn
                                >
                            </v-col>

                            <v-col cols="6">
                                <v-btn
                                    :color="theme"
                                    rounded="pill"
                                    variant="flat"
                                    block
                                    @click="
                                        postFormDelete(
                                            () => (isActive.value = false)
                                        )
                                    "
                                    >HAPUS</v-btn
                                >
                            </v-col>
                        </v-row>
                    </template>
                </form-confirm>

                <v-tooltip activator="parent" location="bottom"
                    >Hapus</v-tooltip
                >
            </v-btn>
        </template>

        <v-btn v-if="withHelpdesk" icon @click="helpState = !helpState">
            <v-icon
                :style="
                    helpState
                        ? 'transform: rotate(180deg)'
                        : 'transform: rotate(0deg)'
                "
                >{{ helpState ? "close" : "menu_open" }}</v-icon
            >

            <v-tooltip activator="parent" location="bottom"
                >Informasi</v-tooltip
            >
        </v-btn>
    </v-app-bar>

    <v-main style="min-height: 100dvh">
        <v-container>
            <page-paper :text-left="title" :max-width="maxWidth">
                <slot
                    :combos="combos"
                    :record="record"
                    :statuses="statuses"
                    :theme="theme"
                    :store="store"
                ></slot>
            </page-paper>
        </v-container>
    </v-main>

    <form-help mode="show" :withActivityLogs="withActivityLogs">
        <template v-slot:feed>
            <slot
                name="feed"
                :combos="combos"
                :theme="theme"
                :record="record"
                :statuses="statuses"
            ></slot>
        </template>

        <template v-slot:info>
            <slot
                name="info"
                :combos="combos"
                :theme="theme"
                :record="record"
                :statuses="statuses"
            ></slot>
        </template>

        <template v-slot:icon>
            <slot
                name="icon"
                :combos="combos"
                :theme="theme"
                :record="record"
                :statuses="statuses"
            ></slot>
        </template>
    </form-help>
</template>

<script>
import { usePageStore } from "@pinia/pageStore";
import { storeToRefs } from "pinia";

export default {
    name: "form-show",

    props: {
        beforePost: Function,
        dataFromStore: Boolean,
        hideEdit: Boolean,
        hideDelete: Boolean,

        maxWidth: {
            type: String,
            default: "560px",
        },

        navbackTo: String,
        withActivityLogs: Boolean,
        withHelpdesk: Boolean,
    },

    setup(props) {
        const store = usePageStore();

        store.beforePost = props.beforePost;
        store.activityLog = props.withActivityLogs;

        const {
            canEdit,
            canDelete,
            canRestore,
            canDestroy,

            combos,
            helpState,
            highlight,
            key,
            page,
            pageKey,
            softdelete,
            statuses,
            record,
            theme,
            title,
        } = storeToRefs(store);

        const {
            getPageData,
            openFormData,
            openFormEdit,
            postFormDelete,
            postFormForceDelete,
            postFormRestore,
        } = store;

        return {
            canEdit,
            canDelete,
            canRestore,
            canDestroy,

            combos,
            helpState,
            highlight,
            key,
            page,
            pageKey,
            record,
            softdelete,
            statuses,
            theme,
            title,

            getPageData,
            openFormData,
            openFormEdit,
            postFormDelete,
            postFormForceDelete,
            postFormRestore,

            store,
        };
    },

    created() {
        this.getPageData(this.dataFromStore);
    },
};
</script>
