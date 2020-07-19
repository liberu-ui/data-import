<template>
    <enso-select v-model="param.value"
        :options="options"
        :multiple="param.multiple"/>
</template>

<script>
import { EnsoSelect } from '@enso-ui/select/bulma';

export default {
    name: 'Enum',

    components: { EnsoSelect },

    inject: ['i18n'],

    props: {
        param: {
            type: Object,
            required: true,
        },
    },

    computed: {
        options() {
            const column = this.state.template.columns
                .find(({ data }) => data === this.param.data);

            return column.enum._select();
        },
    },

    created() {
        this.param.type = 'enum';
        this.param.value = [];
    },
};
</script>
