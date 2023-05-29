<template>
    <span>
        <template v-if="field.value && field.value.length">
            <Link
                v-for="(resource, index) in field.value"
                :key="index"
                :component="'detail'"
                :href="`/nova/resources/${field.resourceName}/${resource.id}`"
                :data="{
                    resourceName: field.resourceName,
                    resourceId: resource.id,
                }"
                class="dim text-primary font-bold no-underline"
            >
                {{ resource.text }} {{ field.value.length - index - 1 ? ' , ' : '' }}
            </Link>
        </template>
        <!-- <template v-else><p #value >&mdash;</p></template> -->
    </span>
</template>

<script>
import { Link } from '@inertiajs/inertia-vue3';
export default {
    components: { Link },
    props: ['resourceName', 'field'],
};
</script>
