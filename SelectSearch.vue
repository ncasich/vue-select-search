<template>
    <select class="form-control" ref="select" v-model="value" :disabled="disabled" :multiple="multiple">
        <slot>
            <template v-if="!!options && options.length">
                <option v-for="option in options" :key="option.id" :value="option.id">{{ option.name }}</option>
            </template>
        </slot>
    </select>
</template>

<script>
    require('select2/dist/js/select2.full.js');
    require('select2/dist/css/select2.min.css');
    require('select2-bootstrap-theme/dist/select2-bootstrap.min.css');

    export default {
        name: "select-search",
        props: ['value', 'holder', 'disabled', 'multiple', 'options'],
        events: ['input', 'change', 'changing', 'opening', 'closing', 'clearing'],
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
                this.select = $(this.$refs.select).select2(this.properties)
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
                this.$emit('input', this.parseValue(e));
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
            string(val) {
                return [...val].sort().join(",");
            },
        },
        computed: {
            properties() {
                let props = {width: '100%', theme: "bootstrap"};
                if (this.holder) {
                    props.placeholder = this.holder;
                }
                return props;
            }
        },
        mounted() {
            this.init();
        },
        watch: {
            value(val) {
                try {
                    if (this.select && (!this.multiple || this.string(val) !== this.string($(this.$el).val()))) {
                        this.select.val(val).trigger('change');
                    }
                } catch (e) {
                }
            },
            options(val) {
                this.init();
            },
        },
    }
</script>