<template>
    <enso-select v-model="param.value"
         :source="param.route"
         :options="param.options"
         :label="param.selectLabel"
         :placeholder="placeholder"
         :multiple="param.multiple"
         :params="param.params"/>
</template>

<script>
import { EnsoSelect } from '@liberu-ui/select/bulma';

export default {
    name: 'Select',

    components: { EnsoSelect },

    inject: ['i18n'],

    props: {
        param: {
            type: Object,
            required: true,
        },
    },

    computed: {
        placeholder() {
            return this.param.placeholder
                ? this.i18n(this.param.placeholder)
                : this.param.placeholder;
        },
    },
};
</script>
