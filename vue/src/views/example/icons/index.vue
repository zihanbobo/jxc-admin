<template>
    <el-card>
        <div class="tip-row">
            <a href="https://panjiachen.github.io/vue-element-admin-site/guide/advanced/icon.html" target="_blank">
                Add and use
            </a>
        </div>
        <el-tabs type="border-card">
            <el-tab-pane label="svg-icons">
                <div :key="item" class="icon-item" v-for="item of svgIcons">
                    <svg-icon :icon="item"/>
                    <span>{{ item }}</span>
                </div>
            </el-tab-pane>
            <el-tab-pane label="element-icons">
                <div :key="item" class="icon-item" v-for="item of elementIcons">
                    <i :class="item"/>
                    <span>{{ item.replace('el-icon-','') }}</span>
                </div>
            </el-tab-pane>
        </el-tabs>
    </el-card>
</template>

<script>
    import svgIcons from '@/assets/icons'
    import elementIcons from './element-icons'

    export default {
        name: "icons",
        data() {
            return {
                svgIcons,
                elementIcons
            }
        },
        methods: {
            generateIconCode(symbol) {
                return `<svg-icon icon="${symbol}" />`
            },
            generateElementIconCode(symbol) {
                return `<i class="${symbol}" />`
            },
        }
    }
</script>

<style scoped>
    .icon-item {
        margin: 20px;
        height: 85px;
        text-align: center;
        width: 100px;
        float: left;
        font-size: 30px;
        color: #24292e;
        cursor: pointer;
    }

    span {
        display: block;
        font-size: 16px;
        margin-top: 10px;
    }
</style>
