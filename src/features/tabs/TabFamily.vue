<template>
    <div
        v-if="unref(visibility) !== Visibility.None"
        class="tab-family-container"
        :class="{ ...unref(classes), ...tabClasses }"
        :style="[
            {
                visibility: unref(visibility) === Visibility.Hidden ? 'hidden' : undefined
            },
            unref(style) ?? [],
            tabStyle ?? []
        ]"
    >
        <Sticky class="tab-buttons-container">
            <div class="tab-buttons" :class="{ floating }">
                <TabButton
                    v-for="(button, id) in unref(tabs)"
                    @selectTab="selected.value = id"
                    :floating="floating"
                    :key="id"
                    :active="unref(button.tab) === unref(activeTab)"
                    v-bind="gatherButtonProps(button)"
                />
            </div>
        </Sticky>
        <template v-if="unref(activeTab)">
            <component :is="unref(component)" />
        </template>
    </div>
</template>

<script lang="ts">
import Sticky from "components/layout/Sticky.vue";
import themes from "data/themes";
import type { CoercableComponent, StyleValue } from "features/feature";
import { Visibility } from "features/feature";
import type { GenericTab } from "features/tabs/tab";
import TabButton from "features/tabs/TabButton.vue";
import type { GenericTabButton } from "features/tabs/tabFamily";
import settings from "game/settings";
import { coerceComponent, isCoercableComponent, processedPropType, unwrapRef } from "util/vue";
import type { Component, PropType, Ref } from "vue";
import { computed, defineComponent, shallowRef, toRefs, unref, watchEffect } from "vue";

export default defineComponent({
    props: {
        visibility: {
            type: processedPropType<Visibility>(Number),
            required: true
        },
        activeTab: {
            type: processedPropType<GenericTab | CoercableComponent | null>(Object),
            required: true
        },
        selected: {
            type: Object as PropType<Ref<string>>,
            required: true
        },
        tabs: {
            type: processedPropType<Record<string, GenericTabButton>>(Object),
            required: true
        },
        style: processedPropType<StyleValue>(String, Object, Array),
        classes: processedPropType<Record<string, boolean>>(Object)
    },
    components: {
        Sticky,
        TabButton
    },
    setup(props) {
        const { activeTab } = toRefs(props);

        const floating = computed(() => {
            return themes[settings.theme].floatingTabs;
        });

        const component = shallowRef<Component | string>("");

        watchEffect(() => {
            const currActiveTab = unwrapRef(activeTab);
            if (currActiveTab == null) {
                component.value = "";
                return;
            }
            if (isCoercableComponent(currActiveTab)) {
                component.value = coerceComponent(currActiveTab);
                return;
            }
            component.value = coerceComponent(unref(currActiveTab.display));
        });

        const tabClasses = computed(() => {
            const currActiveTab = unwrapRef(activeTab);
            const tabClasses =
                isCoercableComponent(currActiveTab) || !currActiveTab
                    ? undefined
                    : unref(currActiveTab.classes);
            return tabClasses;
        });

        const tabStyle = computed(() => {
            const currActiveTab = unwrapRef(activeTab);
            return isCoercableComponent(currActiveTab) || !currActiveTab
                ? undefined
                : unref(currActiveTab.style);
        });

        function gatherButtonProps(button: GenericTabButton) {
            const { display, style, classes, glowColor, visibility } = button;
            return { display, style: unref(style), classes, glowColor, visibility };
        }

        return {
            floating,
            tabClasses,
            tabStyle,
            Visibility,
            component,
            gatherButtonProps,
            unref
        };
    }
});
</script>

<style scoped>
.tab-family-container {
    margin: calc(50px + var(--feature-margin)) 20px var(--feature-margin) 20px;
    position: relative;
    border: solid 4px;
    border-color: var(--outline);
}

.layer-tab > .tab-family-container:first-child {
    margin: -4px -11px var(--feature-margin) -11px;
}

.layer-tab > .tab-family-container:first-child:nth-last-child(3) {
    border-bottom-style: none;
    border-left-style: none;
    border-right-style: none;
    height: calc(100% + 50px);
}

.modal-body > .tab-family-container:first-child {
    margin: calc(10px + var(--feature-margin)) 10px 0 10px;
    border: none;
}

.tab-family-container > :nth-child(2) {
    margin-top: 20px;
}

.modal-body > .tab-family-container > :nth-child(2) {
    /* TODO Why does it need this instead of 20px? */
    margin-top: 50px;
}

.tab-family-container[data-v-f18896fc] > :last-child {
    margin-bottom: 20px;
}

.tab-buttons-container {
    z-index: 4;
}

.tab-buttons-container:not(.floating) {
    border-bottom: solid 4px;
    border-color: inherit;
}

:not(.layer-tab):not(.modal-body) > .tab-family-container > .tab-buttons-container:not(.floating) {
    width: calc(100% + 6px);
    margin-left: -3px;
}

.tab-buttons-container:not(.floating) .tab-buttons {
    text-align: left;
    margin-bottom: -4px;
}

.tab-buttons-container.floating .tab-buttons {
    justify-content: center;
    margin-top: -25px;
}

.tab-buttons {
    margin-bottom: 24px;
    display: flex;
    flex-flow: wrap;
    z-index: 4;
}

.layer-tab
    > .tab-family-container:first-child:nth-last-child(3)
    > .tab-buttons-container
    > .tab-buttons {
    padding-right: 60px;
}

.tab-buttons:not(.floating) {
    text-align: left;
    border-bottom: inherit;
    border-width: 4px;
    box-sizing: border-box;
    height: 50px;
}

.modal-body .tab-buttons {
    width: 100%;
    margin-left: 0;
    margin-right: 0;
    padding-left: 0;
}

.showGoBack
    > .tab-family-container
    > .tab-buttons-container:not(.floating):first-child
    .tab-buttons {
    padding-left: 70px;
}

:not(.showGoBack)
    > .tab-family-container
    > .tab-buttons-container:not(.floating):first-child
    .tab-buttons {
    padding-left: 0;
}

.minimizable > .tab-buttons-container:not(.floating):first-child {
    padding-right: 50px;
}

.tab-buttons-container:not(.floating):first-child .tab-buttons {
    margin-top: -50px;
}

.tab-buttons-container + * {
    margin-top: 20px;
}
</style>
