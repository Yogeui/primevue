<template>
    <div ref="container" :class="containerClass" role="group" :aria-labelledby="ariaLabelledby" v-bind="ptm('root')">
        <div
            v-for="(option, i) of options"
            :key="getOptionRenderKey(option)"
            v-ripple
            :tabindex="i === focusedIndex ? '0' : '-1'"
            :aria-label="getOptionLabel(option)"
            :role="multiple ? 'checkbox' : 'radio'"
            :aria-checked="isSelected(option)"
            :aria-disabled="optionDisabled"
            :class="getButtonClass(option, i)"
            @click="onOptionSelect($event, option, i)"
            @keydown="onKeydown($event, option, i)"
            @focus="onFocus($event)"
            @blur="onBlur($event, option)"
            v-bind="ptm('button')"
        >
            <slot name="option" :option="option" :index="i">
                <span class="p-button-label" v-bind="ptm('label')">{{ getOptionLabel(option) }}</span>
            </slot>
        </div>
    </div>
</template>

<script>
import BaseComponent from 'primevue/basecomponent';
import Ripple from 'primevue/ripple';
import { DomHandler, ObjectUtils } from 'primevue/utils';

export default {
    name: 'SelectButton',
    extends: BaseComponent,
    emits: ['update:modelValue', 'focus', 'blur', 'change'],
    props: {
        modelValue: null,
        options: Array,
        optionLabel: null,
        optionValue: null,
        optionDisabled: null,
        multiple: Boolean,
        unselectable: {
            type: Boolean,
            default: false
        },
        disabled: Boolean,
        dataKey: null,
        'aria-labelledby': {
            type: String,
            default: null
        }
    },
    data() {
        return {
            focusedIndex: 0
        };
    },
    mounted() {
        this.defaultTabIndexes();
    },
    methods: {
        defaultTabIndexes() {
            let opts = DomHandler.find(this.$refs.container, '.p-button');
            let firstHighlight = DomHandler.findSingle(this.$refs.container, '.p-highlight');

            for (let i = 0; i < opts.length; i++) {
                if ((DomHandler.hasClass(opts[i], 'p-highlight') && ObjectUtils.equals(opts[i], firstHighlight)) || (firstHighlight === null && i == 0)) {
                    this.focusedIndex = i;
                }
            }
        },
        getOptionLabel(option) {
            return this.optionLabel ? ObjectUtils.resolveFieldData(option, this.optionLabel) : option;
        },
        getOptionValue(option) {
            return this.optionValue ? ObjectUtils.resolveFieldData(option, this.optionValue) : option;
        },
        getOptionRenderKey(option) {
            return this.dataKey ? ObjectUtils.resolveFieldData(option, this.dataKey) : this.getOptionLabel(option);
        },
        isOptionDisabled(option) {
            return this.optionDisabled ? ObjectUtils.resolveFieldData(option, this.optionDisabled) : false;
        },
        onOptionSelect(event, option, index) {
            if (this.disabled || this.isOptionDisabled(option)) {
                return;
            }

            let selected = this.isSelected(option);

            if (selected && this.unselectable) {
                return;
            }

            let optionValue = this.getOptionValue(option);
            let newValue;

            if (this.multiple) {
                if (selected) newValue = this.modelValue.filter((val) => !ObjectUtils.equals(val, optionValue, this.equalityKey));
                else newValue = this.modelValue ? [...this.modelValue, optionValue] : [optionValue];
            } else {
                newValue = selected ? null : optionValue;
            }

            this.focusedIndex = index;
            this.$emit('update:modelValue', newValue);
            this.$emit('change', { event: event, value: newValue });
        },
        isSelected(option) {
            let selected = false;
            let optionValue = this.getOptionValue(option);

            if (this.multiple) {
                if (this.modelValue) {
                    for (let val of this.modelValue) {
                        if (ObjectUtils.equals(val, optionValue, this.equalityKey)) {
                            selected = true;
                            break;
                        }
                    }
                }
            } else {
                selected = ObjectUtils.equals(this.modelValue, optionValue, this.equalityKey);
            }

            return selected;
        },
        onKeydown(event, option, index) {
            switch (event.code) {
                case 'Space': {
                    this.onOptionSelect(event, option, index);
                    event.preventDefault();
                    break;
                }

                case 'ArrowDown':

                case 'ArrowRight': {
                    this.changeTabIndexes(event, 'next');
                    event.preventDefault();
                    break;
                }

                case 'ArrowUp':

                case 'ArrowLeft': {
                    this.changeTabIndexes(event, 'prev');
                    event.preventDefault();
                    break;
                }

                default:
                    //no op
                    break;
            }
        },
        changeTabIndexes(event, direction) {
            let firstTabableChild, index;

            for (let i = 0; i <= this.$refs.container.children.length - 1; i++) {
                if (this.$refs.container.children[i].getAttribute('tabindex') === '0') firstTabableChild = { elem: this.$refs.container.children[i], index: i };
            }

            if (direction === 'prev') {
                if (firstTabableChild.index === 0) index = this.$refs.container.children.length - 1;
                else index = firstTabableChild.index - 1;
            } else {
                if (firstTabableChild.index === this.$refs.container.children.length - 1) index = 0;
                else index = firstTabableChild.index + 1;
            }

            this.focusedIndex = index;
            this.$refs.container.children[index].focus();
        },
        onFocus(event) {
            this.$emit('focus', event);
        },
        onBlur(event, option) {
            if (event.target && event.relatedTarget && event.target.parentElement !== event.relatedTarget.parentElement) {
                this.defaultTabIndexes();
            }

            this.$emit('blur', event, option);
        },
        getButtonClass(option) {
            return [
                'p-button p-component',
                {
                    'p-highlight': this.isSelected(option),
                    'p-disabled': this.isOptionDisabled(option)
                }
            ];
        }
    },
    computed: {
        containerClass() {
            return [
                'p-selectbutton p-buttonset p-component',
                {
                    'p-disabled': this.disabled
                }
            ];
        },
        equalityKey() {
            return this.optionValue ? null : this.dataKey;
        }
    },
    directives: {
        ripple: Ripple
    }
};
</script>
