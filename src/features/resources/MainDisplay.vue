<template>
    <div>
        <span v-if="showPrefix">You have </span>
        <ResourceVue :resource="resource" :color="color || 'white'" />
        {{ resource.displayName
        }}<!-- remove whitespace -->
        <span v-if="effectComponent">, <component :is="effectComponent" /></span>
        <br /><br />
    </div>
</template>

<script setup lang="ts">
import type { CoercableComponent } from "features/feature";
import type { Resource } from "features/resources/resource";
import ResourceVue from "features/resources/Resource.vue";
import Decimal from "util/bignum";
import { computeOptionalComponent } from "util/vue";
import type { Ref, StyleValue } from "vue";
import { computed, toRefs } from "vue";

const _props = defineProps<{
    resource: Resource;
    color?: string;
    classes?: Record<string, boolean>;
    style?: StyleValue;
    effectDisplay?: CoercableComponent;
}>();
const props = toRefs(_props);

const effectComponent = computeOptionalComponent(
    props.effectDisplay as Ref<CoercableComponent | undefined>
);

const showPrefix = computed(() => {
    return Decimal.lt(props.resource.value, "1e1000");
});
</script>
