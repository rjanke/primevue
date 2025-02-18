<template>
    <ul class="p-submenu-list">
        <template v-for="(processedItem, index) of items" :key="getItemKey(processedItem)">
            <li
                v-if="isItemVisible(processedItem) && !getItemProp(processedItem, 'separator')"
                :id="getItemId(processedItem)"
                :style="getItemProp(processedItem, 'style')"
                :class="getItemClass(processedItem)"
                role="treeitem"
                :aria-label="getItemLabel(processedItem)"
                :aria-expanded="isItemGroup(processedItem) ? isItemActive(processedItem) : undefined"
                :aria-level="level + 1"
                :aria-setsize="getAriaSetSize()"
                :aria-posinset="getAriaPosInset(index)"
                @click="onItemClick($event, processedItem)"
            >
                <div class="p-menuitem-content">
                    <template v-if="!template">
                        <router-link v-if="getItemProp(processedItem, 'to') && !isItemDisabled(processedItem)" v-slot="{ navigate, href, isActive, isExactActive }" :to="getItemProp(processedItem, 'to')" custom>
                            <a v-ripple :href="href" :class="getItemActionClass(processedItem, { isActive, isExactActive })" tabindex="-1" :aria-hidden="true" @click="onItemActionClick($event, navigate)">
                                <span v-if="getItemProp(processedItem, 'icon')" :class="getItemIconClass(processedItem)"></span>
                                <span class="p-menuitem-text">{{ getItemLabel(processedItem) }}</span>
                            </a>
                        </router-link>
                        <a v-else v-ripple :href="getItemProp(processedItem, 'url')" :class="getItemActionClass(processedItem)" :target="getItemProp(processedItem, 'target')" tabindex="-1" :aria-hidden="true">
                            <span v-if="isItemGroup(processedItem)" :class="getItemToggleIconClass(processedItem)"></span>
                            <span v-if="getItemProp(processedItem, 'icon')" :class="getItemIconClass(processedItem)"></span>
                            <span class="p-menuitem-text">{{ getItemLabel(processedItem) }}</span>
                        </a>
                    </template>
                    <component v-else :is="template" :item="processedItem.item"></component>
                </div>
                <transition name="p-toggleable-content">
                    <div v-show="isItemActive(processedItem)" class="p-toggleable-content">
                        <PanelMenuSub
                            v-if="isItemVisible(processedItem) && isItemGroup(processedItem)"
                            :id="getItemId(processedItem) + '_list'"
                            role="group"
                            :panelId="panelId"
                            :focusedItemId="focusedItemId"
                            :items="processedItem.items"
                            :level="level + 1"
                            :template="template"
                            :activeItemPath="activeItemPath"
                            :exact="exact"
                            @item-toggle="onItemToggle"
                        />
                    </div>
                </transition>
            </li>
            <li v-if="isItemVisible(processedItem) && getItemProp(processedItem, 'separator')" :style="getItemProp(processedItem, 'style')" :class="getSeparatorItemClass(processedItem)" role="separator"></li>
        </template>
    </ul>
</template>

<script>
import Ripple from 'primevue/ripple';
import { ObjectUtils } from 'primevue/utils';

export default {
    name: 'PanelMenuSub',
    emits: ['item-toggle'],
    props: {
        panelId: {
            type: String,
            default: null
        },
        focusedItemId: {
            type: String,
            default: null
        },
        items: {
            type: Array,
            default: null
        },
        level: {
            type: Number,
            default: 0
        },
        template: {
            type: Function,
            default: null
        },
        activeItemPath: {
            type: Object,
            default: null
        },
        exact: {
            type: Boolean,
            default: true
        }
    },
    methods: {
        getItemId(processedItem) {
            return `${this.panelId}_${processedItem.key}`;
        },
        getItemKey(processedItem) {
            return this.getItemId(processedItem);
        },
        getItemProp(processedItem, name) {
            return processedItem && processedItem.item ? ObjectUtils.getItemValue(processedItem.item[name]) : undefined;
        },
        getItemLabel(processedItem) {
            return this.getItemProp(processedItem, 'label');
        },
        isItemActive(processedItem) {
            return this.activeItemPath.some((path) => path.key === processedItem.key);
        },
        isItemVisible(processedItem) {
            return this.getItemProp(processedItem, 'visible') !== false;
        },
        isItemDisabled(processedItem) {
            return this.getItemProp(processedItem, 'disabled');
        },
        isItemFocused(processedItem) {
            return this.focusedItemId === this.getItemId(processedItem);
        },
        isItemGroup(processedItem) {
            return ObjectUtils.isNotEmpty(processedItem.items);
        },
        isSameMenuItem(event) {
            return event.currentTarget && (event.currentTarget.isSameNode(event.target) || event.currentTarget.isSameNode(event.target.closest('.p-menuitem')));
        },
        onItemClick(event, processedItem) {
            if (this.isSameMenuItem(event)) {
                const command = this.getItemProp(processedItem, 'command');

                command && command({ originalEvent: event, item: processedItem.item });
                this.$emit('item-toggle', { processedItem, expanded: !this.isItemActive(processedItem) });
            }
        },
        onItemToggle(event) {
            this.$emit('item-toggle', event);
        },
        onItemActionClick(event, navigate) {
            navigate && navigate(event);
        },
        getAriaSetSize() {
            return this.items.filter((processedItem) => this.isItemVisible(processedItem) && !this.getItemProp(processedItem, 'separator')).length;
        },
        getAriaPosInset(index) {
            return index - this.items.slice(0, index).filter((processedItem) => this.isItemVisible(processedItem) && this.getItemProp(processedItem, 'separator')).length + 1;
        },
        getItemClass(processedItem) {
            return [
                'p-menuitem',
                this.getItemProp(processedItem, 'class'),
                {
                    'p-focus': this.isItemFocused(processedItem),
                    'p-disabled': this.isItemDisabled(processedItem)
                }
            ];
        },
        getItemActionClass(processedItem, routerProps) {
            return [
                'p-menuitem-action',
                {
                    'router-link-active': routerProps && routerProps.isActive,
                    'router-link-active-exact': this.exact && routerProps && routerProps.isExactActive
                }
            ];
        },
        getItemIconClass(processedItem) {
            return ['p-menuitem-icon', this.getItemProp(processedItem, 'icon')];
        },
        getItemToggleIconClass(processedItem) {
            return ['p-submenu-icon', this.isItemActive(processedItem) ? 'pi pi-fw pi-chevron-down' : 'pi pi-fw pi-chevron-right'];
        },
        getSeparatorItemClass(processedItem) {
            return ['p-menuitem-separator', this.getItemProp(processedItem, 'class')];
        }
    },
    directives: {
        ripple: Ripple
    }
};
</script>
