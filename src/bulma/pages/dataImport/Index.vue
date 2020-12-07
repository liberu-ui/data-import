<template>
    <div class="wrapper">
        <div class="columns is-variable is-2 is-mobile is-multiline">
            <div class="column is-3-desktop is-half-touch">
                <enso-select v-model="type"
                    :options="enums.importTypes._select()"
                    placeholder="Import Type"
                    @input="type ? template() : null"/>
            </div>
            <template v-if="type">
                <div v-for="param in params"
                    class="column is-3-desktop is-half-touch"
                    :key="param.name">
                    <slot :name="param.name"
                        v-if="param.type === 'slot'"/>
                    <Param :param="param"
                        v-else/>
                </div>
                <div class="column"/>
                <div class="column is-narrow">
                    <div class="is-flex">
                        <a class="button is-info animated fadeIn has-margin-right-medium"
                            :href="templateLink"
                            v-if="type">
                            <span>{{ i18n('Template') }}</span>
                            <span class="icon is-small">
                                <fa icon="download"/>
                            </span>
                        </a>
                        <uploader :url="importLink"
                            :params="uploadParams"
                            :file-size-limit="100000000"
                            file-key="import"
                            @upload-start="loading = true"
                            @upload-error="loading = false"
                            @upload-successful="uploaded"
                            v-if="!hasErrors">
                            <template v-slot:control="{ controlEvents }">
                                <a :class="['button is-success', { 'is-loading': loading }]"
                                    v-on="controlEvents">
                                    <slot>
                                        <span>{{ i18n('Import') }}</span>
                                        <span class="icon is-small">
                                            <fa icon="upload"/>
                                        </span>
                                    </slot>
                                </a>
                            </template>
                        </uploader>
                    </div>
                </div>
            </template>
        </div>
        <enso-table class="box is-paddingless raises-on-hover"
            id="dataImports"
            :filters="filters"
            @download-rejected="rejected"
            ref="imports">
            <template v-slot:type="{ column, row }">
                <span class="tag is-table-tag is-info">
                    {{ column.enum._get(row.type) }}
                </span>
            </template>
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
                <span :class="['tag is-table-tag', enums.importCssClasses._get(row.status)]">
                    {{ column.enum._get(row.status) }}
                </span>
            </template>
            <template v-slot:createdBy="{ column, row }">
                <avatar class="is-24x24"
                    :user="row.createdBy"/>
            </template>
        </enso-table>
        <Summary :summary="summary"
            @close="summary = null"
            v-if="hasErrors"/>
    </div>
</template>

<script>
import { mapState } from 'vuex';
import { library } from '@fortawesome/fontawesome-svg-core';
import {
    faDownload, faUpload, faTrashAlt, faFileExcel, faBan, faSync,
} from '@fortawesome/free-solid-svg-icons';
import { EnsoTable } from '@enso-ui/tables/bulma';
import { EnsoSelect } from '@enso-ui/select/bulma';
import { Avatar } from '@enso-ui/users';
import { Uploader } from '@enso-ui/uploader/bulma';
import Summary from './components/Summary.vue';
import Param from './components/Param.vue';

library.add(faDownload, faTrashAlt, faFileExcel, faBan, faSync);

export default {
    name: 'Index',

    inject: ['canAccess', 'errorHandler', 'i18n', 'route'],

    components: {
        Avatar,
        EnsoSelect,
        EnsoTable,
        Uploader,
        Summary,
        Param,
    },

    data: () => ({
        loading: false,
        type: null,
        summary: null,
        params: [],
    }),

    computed: {
        ...mapState(['enums']),
        filters() {
            return { data_imports: { type: this.type } };
        },
        hasErrors() {
            return this.summary
                && this.summary.errors
                && Object.keys(this.summary.errors).length > 0;
        },
        importLink() {
            return this.canAccess('import.store')
                && this.type
                && this.route('import.store');
        },
        templateLink() {
            return this.canAccess('import.template')
                && this.type
                && this.route('import.template', this.type);
        },
        uploadParams() {
            return this.params.reduce((params, param) => {
                params[param.name] = param.value;
                return params;
            }, { type: this.type });
        },
    },

    methods: {
        template() {
            axios.get(this.route('import.show', this.type))
                .then(({ data: { params } }) => (this.params = params))
                .catch(this.errorHandler);
        },
        rejected({ rejectedId }) {
            window.location.href = this.route('import.rejected', rejectedId);
        },
        uploaded($event) {
            this.summary = $event;
            this.loading = false;
        },
    },
};
</script>
