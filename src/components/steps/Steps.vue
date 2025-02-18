<template>
    <nav :id="id" :class="containerClass">
        <ol ref="list">
            <template v-for="(item, index) of model" :key="item.to">
                <li v-if="visible(item)" :class="getItemClass(item)" :style="item.style">
                    <template v-if="!$slots.item">
                        <router-link v-if="!isItemDisabled(item)" v-slot="{ navigate, href, isActive, isExactActive }" :to="item.to" custom>
                            <a
                                :href="href"
                                :class="linkClass({ isActive, isExactActive })"
                                role="presentation"
                                :tabindex="-1"
                                :aria-current="isExactActive ? 'step' : undefined"
                                @click="onItemClick($event, item, navigate)"
                                @keydown="onItemKeydown($event, item, navigate)"
                            >
                                <span class="p-steps-number">{{ index + 1 }}</span>
                                <span class="p-steps-title">{{ label(item) }}</span>
                            </a>
                        </router-link>
                        <span v-else :class="linkClass()" role="presentation" @keydown="onItemKeydown($event, item)">
                            <span class="p-steps-number">{{ index + 1 }}</span>
                            <span class="p-steps-title">{{ label(item) }}</span>
                        </span>
                    </template>
                    <component v-else :is="$slots.item" :item="item"></component>
                </li>
            </template>
        </ol>
    </nav>
</template>

<script>
import { DomHandler, UniqueComponentId } from 'primevue/utils';

export default {
    name: 'Steps',
    props: {
        id: {
            type: String,
            default: UniqueComponentId()
        },
        model: {
            type: Array,
            default: null
        },
        readonly: {
            type: Boolean,
            default: true
        },
        exact: {
            type: Boolean,
            default: true
        }
    },
    mounted() {
        const firstItem = this.findFirstItem();

        firstItem.tabIndex = '0';
    },
    methods: {
        onItemClick(event, item, navigate) {
            if (this.disabled(item) || this.readonly) {
                event.preventDefault();

                return;
            }

            if (item.command) {
                item.command({
                    originalEvent: event,
                    item: item
                });
            }

            if (item.to && navigate) {
                navigate(event);
            }
        },
        onItemKeydown(event, item, navigate) {
            switch (event.code) {
                case 'ArrowRight': {
                    this.navigateToNextItem(event.target);
                    event.preventDefault();
                    break;
                }

                case 'ArrowLeft': {
                    this.navigateToPrevItem(event.target);
                    event.preventDefault();
                    break;
                }

                case 'Home': {
                    this.navigateToFirstItem(event.target);
                    event.preventDefault();
                    break;
                }

                case 'End': {
                    this.navigateToLastItem(event.target);
                    event.preventDefault();
                    break;
                }

                case 'Tab':
                    //no op
                    break;

                case 'Enter':

                case 'Space': {
                    this.onItemClick(event, item, navigate);
                    event.preventDefault();
                    break;
                }

                default:
                    break;
            }
        },
        navigateToNextItem(target) {
            const nextItem = this.findNextItem(target);

            nextItem && this.setFocusToMenuitem(target, nextItem);
        },
        navigateToPrevItem(target) {
            const prevItem = this.findPrevItem(target);

            prevItem && this.setFocusToMenuitem(target, prevItem);
        },
        navigateToFirstItem(target) {
            const firstItem = this.findFirstItem(target);

            firstItem && this.setFocusToMenuitem(target, firstItem);
        },
        navigateToLastItem(target) {
            const lastItem = this.findLastItem(target);

            lastItem && this.setFocusToMenuitem(target, lastItem);
        },
        findNextItem(item) {
            const nextItem = item.parentElement.nextElementSibling;

            return nextItem ? nextItem.children[0] : null;
        },
        findPrevItem(item) {
            const prevItem = item.parentElement.previousElementSibling;

            return prevItem ? prevItem.children[0] : null;
        },
        findFirstItem() {
            const firstSibling = DomHandler.findSingle(this.$refs.list, '.p-steps-item');

            return firstSibling ? firstSibling.children[0] : null;
        },
        findLastItem() {
            const siblings = DomHandler.find(this.$refs.list, '.p-steps-item');

            return siblings ? siblings[siblings.length - 1].children[0] : null;
        },
        setFocusToMenuitem(target, focusableItem) {
            target.tabIndex = '-1';
            focusableItem.tabIndex = '0';
            focusableItem.focus();
        },
        isActive(item) {
            return item.to ? this.$router.resolve(item.to).path === this.$route.path : false;
        },
        getItemClass(item) {
            return [
                'p-steps-item',
                item.class,
                {
                    'p-highlight p-steps-current': this.isActive(item),
                    'p-disabled': this.isItemDisabled(item)
                }
            ];
        },
        linkClass(routerProps) {
            return [
                'p-menuitem-link',
                {
                    'router-link-active': routerProps && routerProps.isActive,
                    'router-link-active-exact': this.exact && routerProps && routerProps.isExactActive
                }
            ];
        },
        isItemDisabled(item) {
            return this.disabled(item) || (this.readonly && !this.isActive(item));
        },
        visible(item) {
            return typeof item.visible === 'function' ? item.visible() : item.visible !== false;
        },
        disabled(item) {
            return typeof item.disabled === 'function' ? item.disabled() : item.disabled;
        },
        label(item) {
            return typeof item.label === 'function' ? item.label() : item.label;
        }
    },
    computed: {
        containerClass() {
            return ['p-steps p-component', { 'p-readonly': this.readonly }];
        }
    }
};
</script>

<style>
.p-steps {
    position: relative;
}

.p-steps ol {
    padding: 0;
    margin: 0;
    list-style-type: none;
    display: flex;
}

.p-steps-item {
    position: relative;
    display: flex;
    justify-content: center;
    flex: 1 1 auto;
}

.p-steps-item .p-menuitem-link {
    display: inline-flex;
    flex-direction: column;
    align-items: center;
    overflow: hidden;
    text-decoration: none;
}

.p-steps.p-steps-readonly .p-steps-item {
    cursor: auto;
}

.p-steps-item.p-steps-current .p-menuitem-link {
    cursor: default;
}

.p-steps-title {
    white-space: nowrap;
}

.p-steps-number {
    display: flex;
    align-items: center;
    justify-content: center;
}

.p-steps-title {
    display: block;
}
</style>
