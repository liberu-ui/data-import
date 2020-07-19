<template>
    <div class="columns">
        <div v-for="param in params"
             class="column is-3-desktop is-8-tablet is-12-mobile"
             :key="param.name">
            <div class="field" v-if="param.type !== 'custom'">
                <label class="label"
                    v-if="param.label">
                    {{ i18n(param.label) }}
                </label>
                <component :is="component(param)"
                    :param="param"/>
            </div>
            <slot v-else
                :name="param.name"/>
        </div>
    </div>
</template>

<script>
import { library } from '@fortawesome/fontawesome-svg-core';
import { faFilter } from '@fortawesome/free-solid-svg-icons';
import Boolean from './params/Boolean.vue';
import String from './params/String.vue';
import Date from './params/Date.vue';
import CustomSelect from './params/CustomSelect.vue';

library.add(faFilter);

export default {
    name: 'Params',

    components: {
        Boolean, String, Date, CustomSelect,
    },

    inject: ['route', 'i18n', 'errorHandler'],
    props: {
        params: {
            type: Array,
            required: true,
        },
    },

    data: () => ({
        filter: null,
        isValid: false,
    }),
    computed: {
        custom() {
            return this.state.template.filters?.map((filter) => ({
                ...filter, component: this.component(filter),
            })) || [];
        },
        dynamic() {
            return this.state.template.columns
                .filter(({ meta }) => meta.filterable)
                .map((column) => this.filterFactory(column));
        },
        filters() {
            return [...this.custom, ...this.dynamic];
        },
        hasSelect() {
            return this.filter
                && ['enum', 'select'].includes(this.filter.type);
        },
    },

    methods: {
        component(param) {
            return param.type === 'select'
                ? 'custom-select'
                : param.type;
        },
        slots() {
            return this.params.filter((param) => param.type === 'custom')
                .map((param) => param.name);
        },
    },
};
</script>

<style lang="scss">
</style>
