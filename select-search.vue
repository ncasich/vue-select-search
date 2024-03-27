<template>
    <select class="form-select"
            :ref="id"
            :id="id"
            :disabled="disabled"
            :multiple="multiple ? 'multiple' : false">
        <slot>
            <option v-if="!!options && options.length"
                    v-for="option in options"
                    :key="option.id"
                    :value="option.id">
                {{ option.name }}
            </option>
        </slot>
    </select>
</template>

<script>
import * as select2 from 'select2';
import 'select2/dist/css/select2.min.css'
import 'select2-bootstrap-5-theme/dist/select2-bootstrap-5-theme.min.css';
import {v4 as uuid} from 'uuid';

export default {
    name: "select-search",
    props: {
        modelValue: [String, Array],
        id: {type: String, default: () => 'select-' + uuid()},
        placeholder: {type: String, default: ''},
        options: {type: Array, default: () => []},
        disabled: {type: Boolean, default: false},
        multiple: {type: Boolean, default: false},
        settings: {
            type: Object, default: () => {
            }
        },
    },
    emits: ['update:modelValue', 'change', 'changing', 'opening', 'closing', 'clearing', 'selecting', 'unselecting'],
    data() {
        return {
            select: null,
        };
    },
    methods: {
        init() {
            if (this.select) {
                this.select.select2('destroy');
            }
            this.select = $(this.$refs[this.id]).select2(this.properties)
                .on('select2:selecting', this.selecting)
                .on('select2:unselecting', this.unselecting)
                .on('select2:select', this.input)
                .on('select2:unselect', this.input)
                .on('change', this.change)
                .on('select2:opening', this.opening)
                .on('select2:closing', this.closing)
                .on('select2:clearing', this.clearing)
                .val(this.value)
                .trigger('change');
        },
        opening(e) {
            this.$emit('opening', e, this.parseValue(e));
        },
        input(e) {
            this.$emit('update:modelValue', this.parseValue(e));
        },
        selecting(e) {
            this.$emit('selecting', e, this.parseValue(e));
            this.$emit('changing', e, this.parseValue(e));
        },
        unselecting(e) {
            this.$emit('unselecting', e, this.parseValue(e));
            this.$emit('changing', e, this.parseValue(e));
        },
        changing(e) {
            this.$emit('changing', e, this.parseValue(e));
        },
        change(e) {
            this.$emit('change', e, this.parseValue(e));
        },
        closing(e) {
            this.$emit('closing', e, this.parseValue(e));
        },
        clearing(e) {
            this.$emit('clearing', e, this.parseValue(e));
        },
        parseValue(e) {
            let val = $(e.target).val();
            return this.multiple && !val ? [] : val;
        },
    },
    computed: {
        properties() {
            return {
                width: '100%',
                theme: 'bootstrap-5',
                placeholder: this.placeholder,
                multiple: this.multiple,
                ...this.settings,
            };
        },
        value() {
            return !!this.modelValue && typeof this.modelValue === 'object' ? [...this.modelValue] : this.modelValue;
        }
    },
    mounted() {
        if (typeof select2.default === 'function') {
            select2.default();
        }
        this.init();
    },
    watch: {
        value: {
            handler(val) {
                this.select.val(val).trigger('change');
            },
        },
        options: {
            handler() {
                this.select.empty();
                this.init();
            },
            deep: true,
        },
    },
}
</script>
