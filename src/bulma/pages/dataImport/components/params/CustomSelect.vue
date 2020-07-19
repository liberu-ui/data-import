<template>
    <enso-select v-model="param.value"
         :source="param.route"
         :multiple="param.multiple"
         :params="param.params"/>
</template>

<script>
import { EnsoSelect } from '@enso-ui/select/bulma';

export default {
    name: 'CustomSelect',

    components: { EnsoSelect },

    inject: ['i18n'],

    props: {
        param: {
            type: Object,
            required: true,
        },
    },

    created() {
        this.param.value = this.param.multiple ? [] : null;
    },
};
</script>
