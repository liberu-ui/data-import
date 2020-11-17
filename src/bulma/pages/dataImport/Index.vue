<template>
    <div class="wrapper">
        <div class="columns">
            <div class="column is-3-desktop is-8-tablet is-12-mobile">
                <enso-select v-model="type"
                    :options="types"
                    placeholder="Import Type"
                    @input="getTemplate"/>
            </div>
            <div class="column is-6 is-hidden-touch has-text-centered"
                v-if="type">
                <a class="button is-info animated fadeIn has-margin-right-small"
                    :href="downloadLink">
                    <span>{{ i18n('Download Template') }}</span>
                    <span class="icon is-small">
                        <fa icon="download"/>
                    </span>
                </a>
            </div>
            <div class="column has-text-centered-touch has-text-right-desktop"
                v-if="type">
                <import-uploader class="is-pulled-right"
                    :path="importLink"
                    :params="uploadParams"
                    @upload-successful="$refs.imports.fetch()"/>
            </div>
        </div>
        <params :params="params"
            v-if="type"
            ref="params">
            <template v-for="slot in slots"
                v-slot:[slot]>
                <slot :name="slot"/>
            </template>
        </params>
        <enso-table class="box is-paddingless raises-on-hover"
            id="dataImports"
            :filters="filters"
            @download-rejected="downloadRejected"
            ref="imports">
            <template v-slot:entries="{ row }">
                <strong class="has-text-info">
                    {{ row.entries || '-' }}
                </strong>
            </template>
            <template v-slot:successful="{ row }">
                <strong class="has-text-success">
                    {{ row.successful === null ? '-' : row.successful }}
                </strong>
            </template>
            <template v-slot:failed="{ row }">
                <strong class="has-text-danger">
                    {{ row.failed === null ? '-' : row.failed }}
                </strong>
            </template>
            <template v-slot:status="{ column, row }">
                <span :class="['tag is-table-tag', cssClass(column, row)]">
                    {{ column.enum._get(row.status) }}
                </span>
            </template>
        </enso-table>
        <template-modal :show="summaryModal"
            @close="summaryModal = false"
            @commit="deleteTemplate(template.id); summaryModal = false"/>
    </div>
</template>

<script>
import { VTooltip } from 'v-tooltip';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faUpload, faDownload, faTrashAlt, faFileExcel, faBan,
} from '@fortawesome/free-solid-svg-icons';
import { EnsoTable } from '@enso-ui/tables/bulma';
import { EnsoSelect } from '@enso-ui/select/bulma';
import ImportUploader from './components/ImportUploader.vue';
import TemplateModal from './components/TemplateModal.vue';
import Params from './components/Params.vue';

library.add(
    faUpload, faDownload, faTrashAlt, faFileExcel, faBan,
);

export default {
    name: 'Index',

    inject: ['canAccess', 'errorHandler', 'i18n', 'route', 'toastr'],

    components: {
        EnsoSelect,
        EnsoTable,
        ImportUploader,
        TemplateModal,
        Params,
    },

    directives: { tooltip: VTooltip },

    data: () => ({
        type: null,
        summary: {},
        summaryModal: false,
        loadingTemplate: false,
        types: [],
        vendorId: null,
        params: [],
    }),

    computed: {
        uploadParams() {
            const result = { type: this.type };
            this.params.forEach((param) => {
                result[param.name] = param.value;
            });

            return result;
        },
        filters() {
            return {
                data_imports: { type: this.type },
            };
        },
        downloadLink() {
            return this.canAccess('import.downloadTemplate')
                && this.type
                && this.route('import.downloadTemplate', this.type);
        },
        importLink() {
            return this.canAccess('import.store')
                && this.type
                && this.route('import.store');
        },
        hasErrors() {
            return this.summary
                && this.summary.errors
                && Object.keys(this.summary.errors).length;
        },
        slots() {
            return this.params.filter((param) => param.type === 'custom')
                .map((param) => param.name);
        },
    },

    created() {
        axios.get(this.route('import.index'))
            .then(({ data }) => {
                this.types = data.types;
            }).catch(this.errorHandler);
    },

    methods: {
        cssClass(column, { status }) {
            switch (`${status}`) {
            case column.enum.Waiting:
                return 'is-info';
            case column.enum.Processing:
                return 'is-warning';
            case column.enum.Processed:
                return 'is-primary';
            case column.enum.ExportingRejected:
                return 'is-danger';
            case column.enum.Finalized:
                return 'is-success';
            case column.enum.Canceled:
                return 'is-danger';
            default:
                throw Error;
            }
        },
        getTemplate() {
            if (!this.type) {
                return;
            }

            this.loadingTemplate = true;

            axios.get(this.route('import.show', this.type))
                .then(({ data }) => {
                    this.params = data.import.params;
                    this.loadingTemplate = false;
                }).catch((error) => {
                    this.loadingTemplate = false;
                    this.errorHandler(error);
                });
        },
        downloadRejected({ rejectedId }) {
            if (!rejectedId) {
                this.toastr.info(this.i18n('No rejected summary available'));
                return;
            }

            window.location.href = this.route('import.downloadRejected', rejectedId);
        },
    },
};
</script>
