<template>
    <modal>
        <div class="box">
            <p class="title is-4 has-text-centered">
                <fa icon="exclamation-triangle"/>
                {{ i18n('Structure Errors') }}
            </p>
            <div v-for="(errors, category) in summary.errors"
                :key="category">
                <p class="title is-6 has-text-centered is-marginless">
                    {{ category }}:
                </p>
                <div class="tags is-centered py-3">
                    <span class="tag is-medium is-bold has-text-danger"
                        v-for="error in errors"
                        :key="error">
                        {{ error }}
                    </span>
                </div>
            </div>
        </div>
    </modal>
</template>

<script>
import { FontAwesomeIcon as Fa } from '@fortawesome/vue-fontawesome';
import { library } from '@fortawesome/fontawesome-svg-core';
import { faExclamationTriangle } from '@fortawesome/free-solid-svg-icons';
import { Modal } from '@liberu-ui/modal/bulma';

library.add(faExclamationTriangle);

export default {
    name: 'Summary',

    components: { Fa, Modal },

    inject: ['i18n'],

    props: {
        summary: {
            type: Object,
            required: true,
        },
    },
};
</script>
