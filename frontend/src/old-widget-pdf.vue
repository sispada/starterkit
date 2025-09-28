<script setup>
import { ref, watch, onBeforeUnmount } from "vue";
import * as pdfjsLib from "pdfjs-dist";

pdfjsLib.GlobalWorkerOptions.workerSrc = new URL(
    "pdfjs-dist/build/pdf.worker.mjs",
    import.meta.url
).toString();

defineOptions({
    name: "widget-pdf",
});

const props = defineProps({
    state: { type: Boolean, default: false },
    src: { type: String, required: true },
});

const canvas = ref(null);
let pdfDoc = null;
let renderTask = null;

const currentPage = ref(1);
const totalPages = ref(0);

async function renderPage(num) {
    if (!pdfDoc || !canvas.value) return;

    if (renderTask) {
        renderTask.cancel();
    }

    const page = await pdfDoc.getPage(num);
    const viewport = page.getViewport({ scale: 1.2 });

    const ctx = canvas.value.getContext("2d");
    canvas.value.height = viewport.height;
    canvas.value.width = viewport.width;

    renderTask = page.render({ canvasContext: ctx, viewport });
    await renderTask.promise;

    currentPage.value = num;
}

async function loadPdf() {
    if (!props.src) return;

    const loadingTask = pdfjsLib.getDocument({
        url: props.src,
        withCredentials: true,
    });

    pdfDoc = await loadingTask.promise;
    totalPages.value = pdfDoc.numPages;

    await renderPage(1);
}

function cleanup() {
    if (renderTask) {
        renderTask.cancel();
        renderTask = null;
    }
    if (pdfDoc) {
        pdfDoc.destroy();
        pdfDoc = null;
    }
    if (canvas.value) {
        const ctx = canvas.value.getContext("2d");
        ctx?.clearRect(0, 0, canvas.value.width, canvas.value.height);
    }
    currentPage.value = 1;
    totalPages.value = 0;
}

function nextPage() {
    if (currentPage.value < totalPages.value) {
        renderPage(currentPage.value + 1);
    }
}

function prevPage() {
    if (currentPage.value > 1) {
        renderPage(currentPage.value - 1);
    }
}

// ✅ Download file
function downloadPdf() {
    const link = document.createElement("a");
    link.href = props.src;
    link.setAttribute("download", "document.pdf");
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
}

// ✅ Print file
function printPdf() {
    const win = window.open(props.src, "_blank");
    win?.addEventListener("load", () => {
        win.print();
    });
}

watch(
    () => props.state,
    async (val) => {
        if (val) {
            await loadPdf();
        } else {
            cleanup();
        }
    },
    { immediate: true }
);

onBeforeUnmount(() => {
    cleanup();
});
</script>

<template>
    <v-card>
        <v-toolbar density="comfortable">
            <v-toolbar-title>Pdf Viewer</v-toolbar-title>

            <template v-slot:append>
                <!-- Prev / Next -->
                <v-btn
                    icon="chevron_backward"
                    @click="prevPage"
                    :disabled="currentPage <= 1"
                ></v-btn>
                <span>{{ currentPage }} / {{ totalPages }}</span>
                <v-btn
                    icon="chevron_forward"
                    @click="nextPage"
                    :disabled="currentPage >= totalPages"
                ></v-btn>

                <v-divider vertical></v-divider>

                <!-- Download -->
                <v-btn icon="download" @click="downloadPdf"></v-btn>

                <!-- Print -->
                <v-btn icon="print" @click="printPdf"></v-btn>

                <v-divider vertical></v-divider>

                <!-- Close -->
                <v-btn icon="close" @click="$emit('close')"></v-btn>
            </template>
        </v-toolbar>

        <v-sheet
            class="d-flex flex-column align-center overflow-y-auto"
            height="calc(100dvh - 56px)"
        >
            <canvas ref="canvas"></canvas>
        </v-sheet>
    </v-card>
</template>
