<template>
    <datepicker class="has-margin-top-medium"
        alt-input
        :placeholder="param.label"
        :alt-format="param.meta.dateFormat"
        v-model="param.value"/>
</template>

<script>
import { Datepicker } from '@enso-ui/datepicker/bulma';

export default {
    name: 'Date',

    components: { Datepicker },

    inject: ['i18n'],

    props: {
        param: {
            type: Object,
            required: true,
        },
    },

    created() {
        this.param.type = 'date';
        this.param.meta = this.param.meta || { dateFormat: 'Y-m-d' };
    },

    methods: {
        interval() {
            return {
                min: null,
                max: null,
            };
        },
    },
};
</script>

<style lang="scss">
    .money-filter {
        .level-left, .level-right {
            width: 49%;
            .level-item {
                width: 100%;
            }
        }
    }
</style>
