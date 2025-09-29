<template>
    <v-sheet class="position-relative overflow-hidden" rounded="lg">
        <v-avatar
            :color="color"
            class="position-absolute text-white font-weight-thin"
            style="right: 30px; top: 10px"
        >
            <v-icon>{{ icon }}</v-icon>
        </v-avatar>

        <div class="d-flex">
            <div class="d-flex flex-column flex-grow-1 overflow-hidden">
                <div class="text-overline mx-3 mt-1">{{ title }}</div>

                <div class="flex-grow-1 mx-3 mb-2 d-flex align-end">
                    <div>
                        <span class="text-h5">{{ complete }}</span>
                        <span>/{{ count }}</span>
                    </div>
                </div>
            </div>

            <div
                :class="`bg-${color}-lighten-4`"
                class="d-flex align-end justify-center py-1"
                style="height: 80px; width: 50px"
            >
                <div class="text-center text-caption">{{ year }}</div>
            </div>
        </div>
    </v-sheet>
</template>

<script>
export default {
    name: "widget-statistic",

    props: {
        icon: String,
        title: String,
        complete: String,
        count: String,
        year: String,
        color: String,
    },
};
</script>
