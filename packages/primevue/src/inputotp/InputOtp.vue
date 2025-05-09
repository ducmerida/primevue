<template>
    <div :class="cx('root')" v-bind="ptmi('root')">
        <template v-for="i in length" :key="i">
            <slot :events="getTemplateEvents(i - 1)" :attrs="getTemplateAttrs(i - 1)" :index="i">
                <OtpInputText
                    :value="tokens[i - 1]"
                    :type="inputType"
                    :class="cx('pcInputText')"
                    :name="$formName"
                    :inputmode="inputMode"
                    :variant="variant"
                    :readonly="readonly"
                    :disabled="disabled"
                    :size="size"
                    :invalid="invalid"
                    :tabindex="tabindex"
                    :unstyled="unstyled"
                    @input="onInput($event, i - 1)"
                    @focus="onFocus($event)"
                    @blur="onBlur($event)"
                    @paste="onPaste($event)"
                    @keydown="onKeyDown($event)"
                    @click="onClick($event)"
                    :pt="ptm('pcInputText')"
                />
            </slot>
        </template>
    </div>
</template>

<script>
import { isTouchDevice } from '@primeuix/utils/dom';
import InputText from 'primevue/inputtext';
import BaseInputOtp from './BaseInputOtp.vue';

export default {
    name: 'InputOtp',
    extends: BaseInputOtp,
    inheritAttrs: false,
    emits: ['change', 'focus', 'blur'],
    data() {
        return {
            tokens: []
        };
    },
    watch: {
        modelValue: {
            immediate: true,
            handler(newValue) {
                this.tokens = newValue ? newValue.split('') : new Array(this.length);
            }
        }
    },
    methods: {
        getTemplateAttrs(index) {
            return {
                value: this.tokens[index]
            };
        },
        getTemplateEvents(index) {
            return {
                input: (event) => this.onInput(event, index),
                keydown: (event) => this.onKeyDown(event),
                focus: (event) => this.onFocus(event),
                blur: (event) => this.onBlur(event),
                paste: (event) => this.onPaste(event)
            };
        },
        onInput(event, index) {
            this.tokens[index] = event.target.value;
            this.updateModel(event);

            if (event.inputType === 'deleteContentBackward') {
                this.moveToPrev(event);
            } else if (event.inputType === 'insertText' || event.inputType === 'deleteContentForward' || (isTouchDevice() && event instanceof CustomEvent)) {
                this.moveToNext(event);
            }
        },
        updateModel(event) {
            const newValue = this.tokens.join('');

            this.writeValue(newValue, event);
            this.$emit('change', {
                originalEvent: event,
                value: newValue
            });
        },
        moveToPrev(event) {
            let prevInput = this.findPrevInput(event.target);

            if (prevInput) {
                prevInput.focus();
                prevInput.select();
            }
        },
        moveToNext(event) {
            let nextInput = this.findNextInput(event.target);

            if (nextInput) {
                nextInput.focus();
                nextInput.select();
            }
        },
        findNextInput(element) {
            let nextElement = element.nextElementSibling;

            if (!nextElement) return;

            return nextElement.nodeName === 'INPUT' ? nextElement : this.findNextInput(nextElement);
        },
        findPrevInput(element) {
            let prevElement = element.previousElementSibling;

            if (!prevElement) return;

            return prevElement.nodeName === 'INPUT' ? prevElement : this.findPrevInput(prevElement);
        },
        onFocus(event) {
            event.target.select();
            this.$emit('focus', event);
        },
        onBlur(event) {
            this.$emit('blur', event);
        },
        onClick(event) {
            setTimeout(() => event.target.select(), 1);
        },
        onKeyDown(event) {
            if (event.ctrlKey || event.metaKey) {
                return;
            }

            switch (event.code) {
                case 'ArrowLeft':
                    this.moveToPrev(event);
                    event.preventDefault();

                    break;

                case 'ArrowUp':
                case 'ArrowDown':
                    event.preventDefault();

                    break;

                case 'Backspace':
                    if (event.target.value.length === 0) {
                        this.moveToPrev(event);
                        event.preventDefault();
                    }

                    break;

                case 'ArrowRight':
                    this.moveToNext(event);
                    event.preventDefault();

                    break;

                case 'Enter':
                case 'NumpadEnter':
                case 'Tab':
                    break;

                default:
                    if ((this.integerOnly && !(event.code !== 'Space' && Number(event.key) >= 0 && Number(event.key) <= 9)) || (this.tokens.join('').length >= this.length && event.code !== 'Delete')) {
                        event.preventDefault();
                    }

                    break;
            }
        },
        onPaste(event) {
            let paste = event.clipboardData.getData('text');

            if (paste.length) {
                let pastedCode = paste.substring(0, this.length);

                if (!this.integerOnly || !isNaN(pastedCode)) {
                    this.tokens = pastedCode.split('');
                    this.updateModel(event);
                }
            }

            event.preventDefault();
        }
    },
    computed: {
        inputMode() {
            return this.integerOnly ? 'numeric' : 'text';
        },
        inputType() {
            return this.mask ? 'password' : 'text';
        }
    },
    components: {
        OtpInputText: InputText
    }
};
</script>
