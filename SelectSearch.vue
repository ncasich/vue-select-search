<template>
    <select class="form-control" ref="select" v-model="value" :disabled="disabled" :multiple="multiple">
        <slot></slot>
    </select>
</template>

<script>
    require('select2/dist/js/select2.full.js');

    export default {
        name: "select-search",
        props: ['value', 'holder', 'disabled', 'multiple'],
        events: ['input'],
        data() {
            return {
                select: null,
            };
        },
        mounted() {
            let that = this;
            let props = {width: '100%', theme: "bootstrap"};
            if (that.holder) {
                props.placeholder = that.holder;
            }
            that.select = $(that.$refs.select).select2(props)
                .on('change', function () {
                    that.$emit('input', $(this).val());
                });
        },
        watch: {
            value: function (value) {
                if (this.select) {
                    console.log([...value]);
                    if ([...value].sort().join(",") !== [...$(this.$el).val()].sort().join(",")) {
                        this.select.val(value).trigger('change');
                    }
                }
            },
        },
    }
</script>
