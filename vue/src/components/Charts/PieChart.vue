<template>
    <div :style="{height,width}"/>
</template>

<script>
    import resize from '@/mixins/chartResizeMixin'
    import logic from '@/mixins/chartLogicMixin'

    export default {
        mixins: [resize, logic],
        props: {
            title: String,
            data: Array
        },
        watch: {
            data: {
                deep: true,
                handler(val) {
                    this.init(val)
                }
            }
        },
        methods: {
            init(data) {
                this.chart.setOption({
                    title: {
                        text: this.title,
                        left: 'center',
                        align: 'right'
                    },
                    tooltip: {
                        trigger: 'item',
                        formatter: '{b} : {c} ({d}%)'
                    },
                    series: [
                        {
                            type: 'pie',
                            center: ['50%', '58%'],
                            data
                        }
                    ]
                })
            }
        }
    }
</script>
