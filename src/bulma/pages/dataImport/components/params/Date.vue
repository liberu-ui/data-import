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
};
</script>
