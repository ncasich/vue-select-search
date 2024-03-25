<template>
    <select class="form-select"
            :ref="id"
            :id="id"
            :disabled="disabled"
            :multiple="multiple">
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

import select2 from 'select2';
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
    emits: ['update:modelValue', 'change', 'changing', 'opening', 'closing', 'clearing'],
    data() {
        return {
            select: null,
        };
    },
    methods: {
        init() {
            select2();
            if (this.select) {
                this.select.select2('destroy');
            }

            this.select = $(this.$refs[this.id]).select2(this.properties)
                .on('select2:selecting', this.changing)
                .on('select2:unselecting', this.changing)
                .on('select2:select', this.input)
                .on('select2:unselect', this.input)
                .on('change', this.change)
                .on('select2:opening', this.opening)
                .on('select2:closing', this.closing)
                .on('select2:clearing', this.clearing);
        },
        opening(e) {
            this.$emit('opening', e, this.parseValue(e));
        },
        input(e) {
            this.$emit('update:modelValue', this.parseValue(e));
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
                ...this.settings,
            };
        }
    },
    mounted() {
        this.init();
    },
    watch: {
        modelValue: {
            handler(val) {
                this.select.val(val instanceof Array ? [...val] : [val]).trigger('change');
            },
            deep: true
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
