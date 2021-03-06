<template>
    <div>
        <el-popover ref="popover" :value="showPopover" trigger="manual" width="300">
            <div v-html="step.content" style="padding: 10px"/>
            <div class="guide-popover-action">
                <div class="action-close">
                    <el-button v-show="hasDone" size="small" @click="exit">
                        {{step.doneBtnText}}
                    </el-button>
                    <el-button v-show="!hasDone" size="small" @click="exit">
                        {{step.closeBtnText}}
                    </el-button>
                </div>
                <div class="action-step">
                    <el-button v-show="showPrevBtn" class="prev-btn" size="small" @click="previous">
                        {{step.prevBtnText}}
                    </el-button>
                    <el-button v-show="showNextBtn" class="next-btn" size="small" @click="next">
                        {{step.nextBtnText}}
                    </el-button>
                </div>
            </div>
            <div slot="reference" v-show="showStage" id="guide-stage" :style="stageStyle"/>
        </el-popover>
        <div v-show="showStage" id="guide-highlight-element-cover" :style="stageStyle" @click.prevent.stop/>
        <div v-show="showOverlay" id="guide-overlay"/>
    </div>
</template>

<script>
    /*
* 由driver.js改造而来
* vue里交互型的导航不太行，太多东西做不到，换成引导手册吧
* */
    import {debounce, isEmpty} from "@/utils"

    import {addHighlightClasses, getCalculatedPosition, inAppView, jump, removeHighlightClasses} from "./utils"

    export default {
        name: "Guide",
        data() {
            return {
                steps: [],
                options: {
                    stageBackground: '#ffffff',
                    doneBtnText: '完成',
                    closeBtnText: '关闭',
                    nextBtnText: '下一步',
                    prevBtnText: '上一步',
                },
                beforeExit: null,
                isActive: false,
                showStage: false,
                showOverlay: false,
                currentStep: 0,
                moving: false,
                stageStyle: '',
                highlightedElement: null,
                lastHighlightedElement: null
            }
        },
        computed: {
            showPopover() {
                return !this.moving && this.showStage
            },
            showPrevBtn() {
                return this.step.forceShowPrevBtn || this.currentStep > 0 && this.steps[this.currentStep - 1]
            },
            showNextBtn() {
                return this.step.forceShowNextBtn || this.steps[this.currentStep + 1]
            },
            hasDone() {
                return this.currentStep === this.steps.length - 1
            },
            step() {
                if (!this.isActive) return {}
                if (this.steps.length <= 0) return {}
                if (!this.steps[this.currentStep]) return {}
                return {
                    ...this.options,
                    ...this.steps[this.currentStep],
                    index: this.currentStep
                }
            }
        },
        watch: {
            isActive(v) {
                v ? document.body.classList.add('overflow-hidden') : document.body.classList.remove('overflow-hidden')
            }
        },
        methods: {
            start(index = 0) {
                if (!this.steps || this.steps.length === 0) throw new Error('请传入步骤')

                if (index === 'last') index = this.steps.length - 1
                else if (index === 'first') index = 0

                if (index > this.steps.length - 1 || !this.steps[index]) {
                    console.error('error index:', index, 'error steps:', JSON.parse(JSON.stringify(this.steps)))
                    throw new Error('起始步骤有误')
                }

                this.isActive = true
                this.currentStep = index
                this.showOverlay = false
                this.showStage = false

                this.$_highlight()
                this.showOverlay = true
                this.showStage = true
            },
            exit() {
                if (!this.isActive) return
                let done = this.hasDone
                if (this.beforeExit) {
                    let result = this.beforeExit(done)
                    if (result && result.then) {
                        result.then(() => this.$_clear())
                    }
                    if (result !== false) this.$_clear()
                    return
                }
                this.$_clear()
            },
            previous() {
                if (!this.isActive || this.moving) return
                this.moving = true

                if (this.step.onPrevious) {
                    let result = this.step.onPrevious()
                    if (result && result.then) result.then(() => this.$_movePrevious())
                    else result !== false && this.$_movePrevious()
                }
                else this.$_movePrevious()
            },
            next() {
                if (!this.isActive || this.moving) return
                this.moving = true

                if (this.step.onNext) {
                    let result = this.step.onNext()
                    if (result && result.then) result.then(() => this.$_moveNext())
                    else result !== false && this.$_moveNext()
                }
                else this.$_moveNext()
            },
            resize() {
                if (!this.highlightedElement) return
                this.$_setStageStyle(getCalculatedPosition(this.highlightedElement))
                this.$nextTick(() => this.$refs.popover.updatePopper())
            },
            $_movePrevious() {
                if (!this.isActive) return
                this.currentStep--
                this.$_highlight()
            },
            $_moveNext() {
                if (!this.isActive) return
                this.currentStep++
                this.$_highlight()
            },
            $_clear() {
                if (!this.isActive) return
                this.isActive = false
                this.showOverlay = false
                this.showStage = false
                this.currentStep = 0
                this.steps = []
                this.beforeExit = null
                if (this.highlightedElement) removeHighlightClasses(this.highlightedElement)
                this.highlightedElement = null
                this.lastHighlightedElement = null
            },
            $_highlight() {
                if (isEmpty(this.step.element)) throw new Error(`step${this.currentStep}中element为空`)

                let el = document.querySelector(this.step.element)

                if (!el) throw new Error(`${this.step.element} 没有该元素`)

                if (el === this.highlightedElement) return

                if (this.highlightedElement) removeHighlightClasses(this.highlightedElement)

                this.lastHighlightedElement = this.highlightedElement
                this.highlightedElement = el

                if (inAppView(el)) jump(el)

                this.$_setStageStyle(getCalculatedPosition(el))

                setTimeout(() => {
                    this.$refs.popover.updatePopper()
                    this.moving = false
                }, 300)

                addHighlightClasses(el)
            },
            $_setStageStyle({top, left, right, bottom}) {
                const width = right - left
                const height = bottom - top
                this.stageStyle =
                    `width:${width}px;` +
                    `height:${height}px;` +
                    `top:${top}px;` +
                    `left:${left}px;` +
                    `backgroundColor:${this.step.stageBackground}`
            }
        },
        mounted() {
            this.resize = debounce(this.resize)
            window.addEventListener('resize', this.resize)
        },
        beforeDestroy() {
            window.removeEventListener('resize', this.resize)
        }
    }
</script>

<style lang="scss">
    .guide-highlighted-element {
        z-index: 1002 !important;
    }

    .guide-position-relative {
        position: relative !important;
    }

    .guide-fix-stacking {
        z-index: auto !important;
        opacity: 1.0 !important;
        transform: none !important;
        filter: none !important;
        perspective: none !important;
        transform-style: flat !important;
        will-change: unset !important;
    }

    .guide-popover-action {
        position: relative;
        height: 100%;
        width: 100%;

        .action-close {
            display: inline-block;
        }

        .action-step {
            display: inline-block;
            right: 0;
            position: absolute;

            .prev-btn {
                position: absolute;
                right: 90px;
            }

            .next-btn {

            }
        }
    }

    #guide-stage {
        position: absolute;
        background: white;
        transition: all 0.3s;
        z-index: 1001 !important;
        border-radius: 2px;
        /*background: transparent !important;
        outline: 5000px solid rgba(0, 0, 0, 0.75);*/
    }

    #guide-overlay {
        background: black;
        position: fixed;
        top: 0;
        left: 0;
        bottom: 0;
        right: 0;
        opacity: 0.75;
        z-index: 1000 !important;
    }

    #guide-highlight-element-cover {
        position: absolute;
        background: transparent !important;
        z-index: 1003 !important;
    }
</style>
