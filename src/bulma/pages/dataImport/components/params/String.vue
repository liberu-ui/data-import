<template>
    <input class="input"
        :placeholder="i18n(param.label)"
        v-model="param.value">
</template>

<script>

export default {
    name: 'String',

    inject: ['i18n'],

    props: {
        param: {
            type: Object,
            required: true,
        },
    },
};
</script>

<style lang="scss" >
    .table-filters.dropdown {
        .is-right.icon {
            pointer-events: all;

            &.individual-search-mode {
                right: 0.3em;
            }
        }
    }
</style>
