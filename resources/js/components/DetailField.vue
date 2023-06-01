<template>
    <panel-item :field="field">
        <template v-if="field.value && field.value.length" #value>
            <span v-if="field.pivots">
                <span
                    v-for="(resource, index) in field.value"
                    :key="resource.id"
                    class="dim text-primary cursor-pointer font-bold no-underline"
                    :title="__(':resource Details', { resource: resource.text })"
                    @click="displayPivots(resource)"
                >
                    {{ resource.text }} {{ field.value.length - index - 1 ? ' , ' : '' }}
                </span>
            </span>
            <span v-else>
                <Link
                    v-for="(resource, index) in field.value"
                    :key="index"
                    :href="`/nova/resources/${field.resourceName}/${resource.id}`"
                    :data="{
                        resourceName: field.resourceName,
                        resourceId: resource.id,
                    }"
                    class="dim text-primary font-bold no-underline"
                    :title="__('View')"
                >
                    {{ resource.text }} {{ field.value.length - index - 1 ? ' , ' : '' }}
                </Link>
            </span>
            <modal v-if="field.pivots && display" :show="true" role="dialog" @modal-close="handleClose">
                <loading-view :loading="loading">
                    <card :class="'w-action-fields'">
                        <form-heading-field
                            :field="{
                                asHtml: true,
                                value: display.text,
                            }"
                            class="mb-6 pt-6"
                        />

                        <div class="mb-6 px-6 py-3">
                            <!-- Pivot Fields -->
                            <component
                                :is="'detail-' + field.component"
                                v-for="(field, index) in fields"
                                :key="index"
                                :resource-name="resourceName"
                                :field="field"
                                :via-resource="viaResource"
                                :via-resource-id="viaResourceId"
                                :via-relationship="viaRelationship"
                            />
                        </div>

                        <div class="flex px-6 py-3">
                            <div class="ml-auto flex items-center">
                                <button
                                    class="btn btn-link dim text-80 ml-auto mr-6 cursor-pointer"
                                    type="button"
                                    @click.prevent="handleClose"
                                >
                                    {{ __('Close') }}
                                </button>
                                <Link
                                    :href="`/nova/resources/${field.resourceName}/${display.id}`"
                                    :component="'detail'"
                                    :data="{
                                        resourceName: field.resourceName,
                                        resourceId: display.id,
                                    }"
                                    class="dim text-primary font-bold no-underline"
                                    :title="__('View')"
                                >
                                    {{ __(':resource Details', { resource: display.text }) }}
                                </Link>
                            </div>
                        </div>
                    </card>
                </loading-view>
            </modal>
        </template>
        <!-- <p v-else slot="value">&mdash;</p> -->
    </panel-item>
</template>

<script>
import { Link } from '@inertiajs/inertia-vue3';
export default {
    components: { Link },
    props: ['resource', 'resourceName', 'resourceId', 'field'],

    data() {
        return {
            display: false,
            fields: [],
        };
    },

    computed: {},

    methods: {
        displayPivots(resource) {
            this.display = resource;
            this.loading = false;

            this.getPivotFields();
        },

        /**
         * Get all of the available fields for an attachment.
         */
        async getPivotFields(resource) {
            await Nova.request()
                .get(`/nova-api/armincms/${this.resourceName}/pivot-fields/${this.field.resourceName}`, {
                    params: {
                        resourceId: this.resourceId,
                        relatedId: this.display.id,
                        pivotId: this.display.pivotId,
                    },
                })
                .then(({ data }) => {
                    this.fields = data;

                    _.each(this.fields, (field) => {
                        field.fill = () => '';
                    });
                });
        },

        /**
         * Resolve the component name.
         */
        resolveComponentName(field) {
            return field.prefixComponent ? `detail-${field.component}` : field.component;
        },

        handleClose() {
            this.display = null;

            this.$emit('close');
        },
    },
};
</script>
