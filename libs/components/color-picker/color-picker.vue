<template>
    <div
        v-click-outside="handleClose"
        :class="classes">
        <div
            ref="reference"
            :class="wrapClasses"
            @click="toggleVisible">
            <input
                :name="name"
                :value="currentValue"
                type="hidden">
            <Icon :type="arrowType" :size="arrowSize" :class="arrowClasses"></Icon>
            <div
                ref="input"
                :tabindex="disabled ? undefined : 0"
                :class="inputClasses"
                @keydown.tab="onTab"
                @keydown.esc="onEscape"
                @keydown.up="onArrow"
                @keydown.down="onArrow"
            >
                <div :class="[prefixCls + '-color']">
                    <div
                        v-show="value === '' && !visible"
                        :class="[prefixCls + '-color-empty']">
                        <i :class="[iconPrefixCls,'iconfont',  'bw-quxiao-guanbi-shanchu']"></i>
                    </div>
                    <div
                        v-show="value || visible"
                        :style="displayedColorStyle"></div>
                </div>
            </div>
        </div>
        <transition name="slide">
            <Drop
                v-transfer-dom
                v-show="visible"
                ref="drop"
                :placement="placement"
                :data-transfer="transfer"
                :transfer="transfer"
                :class="dropClasses"
            >
                <transition name="fade">
                    <div
                        v-if="visible"
                        :class="[prefixCls + '-picker']">
                        <div :class="[prefixCls + '-picker-wrapper']">
                            <div :class="[prefixCls + '-picker-panel']">
                                <Saturation
                                    ref="saturation"
                                    v-model="saturationColors"
                                    :focused="visible"
                                    @change="childChange"
                                    @keydown.native.tab="handleFirstTab"
                                ></Saturation>
                            </div>
                            <div
                                v-if="hue"
                                :class="[prefixCls + '-picker-hue-slider']">
                                <Hue
                                    v-model="saturationColors"
                                    @change="childChange"></Hue>
                            </div>
                            <div
                                v-if="alpha"
                                :class="[prefixCls + '-picker-alpha-slider']">
                                <Alpha
                                    v-model="saturationColors"
                                    @change="childChange"></Alpha>
                            </div>
                            <recommend-colors
                                v-if="colors.length"
                                :list="colors"
                                :class="[prefixCls + '-picker-colors']"
                                @picker-color="handleSelectColor"></recommend-colors>
                            <recommend-colors
                                v-if="!colors.length && recommend"
                                :list="recommendedColor"
                                :class="[prefixCls + '-picker-colors']"
                                @picker-color="handleSelectColor"></recommend-colors>
                        </div>
                        <div :class="[prefixCls + '-confirm']">
                            <span :class="confirmColorClasses">
                                <template v-if="editable">
                                    <b-input :value="formatColor" size="small" @on-enter="handleEditColor" @on-blur="handleEditColor"></b-input>
                                </template>
                                <template v-else>{{formatColor}}</template>
                            </span>
                            <b-button
                                ref="clear"
                                :tabindex="0"
                                size="small"
                                @click.native="handleClear"
                                @keydown.enter="handleClear"
                                @keydown.native.esc="closer"
                            >{{'??????'}}</b-button>
                            <b-button
                                ref="ok"
                                :tabindex="0"
                                size="small"
                                type="primary"
                                @click.native="handleSuccess"
                                @keydown.native.tab="handleLastTab"
                                @keydown.enter="handleSuccess"
                                @keydown.native.esc="closer"
                            >{{'??????'}}</b-button>
                        </div>
                    </div>
                </transition>
            </Drop>
        </transition>
    </div>
</template>

<script>
import tinycolor from 'tinycolor2'
import {directive as clickOutside} from 'v-click-outside-x'
import TransferDom from '../../utils/directives/transfer-dom'
import Drop from '../../components/select/Dropdown'
import RecommendColors from './recommend-colors.vue'
import Saturation from './saturation.vue'
import Hue from './hue.vue'
import Alpha from './alpha.vue'
import bInput from '../input'
import bButton from '../button'
import Icon from '../icon'
import {oneOf, prefix} from '../../utils/common'
import Emitter from '../../mixins/emitter'
import Prefixes from './prefixMixin'
import {changeColor, toRGBAString} from './utils'

// import tinycolor from 'tinycolor2'
// import {directive as clickOutside} from 'v-click-outside-x'
// import TransferDom from 'iview/src/directives/transfer-dom'
// import Drop from 'iview/src/components/select/dropdown.vue'
// import RecommendColors from './recommend-colors.vue'
// import Saturation from './saturation.vue'
// import Hue from './hue.vue'
// import Alpha from './alpha.vue'
// import iInput from 'iview/src/components/input/input.vue'
// import Icon from 'iview/src/components/icon/icon.vue'
// import {oneOf} from 'iview/src/utils/assist'
// import Emitter from 'iview/src/mixins/emitter'
// import Prefixes from './prefixMixin'
// import {changeColor, toRGBAString} from './utils'

export default {
    name: `${prefix}color-picker`,

    components: {Drop, RecommendColors, Saturation, Hue, Alpha, bInput, Icon, bButton},

    directives: {clickOutside, TransferDom},

    mixins: [Emitter, Prefixes],

    props: {
        value: {
            type: String,
            default: undefined
        },
        hue: {
            type: Boolean,
            default: true
        },
        alpha: {
            type: Boolean,
            default: false
        },
        recommend: {
            type: Boolean,
            default: false
        },
        format: {
            type: String,
            validator (value) {
                return oneOf(value, ['hsl', 'hsv', 'hex', 'rgb'])
            },
            default: undefined
        },
        colors: {
            type: Array,
            default () {
                return []
            }
        },
        disabled: {
            type: Boolean,
            default: false
        },
        size: {
            validator (value) {
                return oneOf(value, ['small', 'large', 'default'])
            },
            default () {
                return !this.$IVIEW || this.$IVIEW.size === '' ? 'default' : this.$IVIEW.size
            }
        },
        hideDropDown: {
            type: Boolean,
            default: false
        },
        placement: {
            type: String,
            validator (value) {
                return oneOf(value, [
                    'top',
                    'top-start',
                    'top-end',
                    'bottom',
                    'bottom-start',
                    'bottom-end',
                    'left',
                    'left-start',
                    'left-end',
                    'right',
                    'right-start',
                    'right-end'
                ])
            },
            default: 'bottom'
        },
        transfer: {
            type: Boolean,
            default () {
                return !this.$IVIEW || this.$IVIEW.transfer === '' ? false : this.$IVIEW.transfer
            }
        },
        name: {
            type: String,
            default: undefined
        },
        editable: {
            type: Boolean,
            default: true
        }
    },

    data () {
        return {
            val: changeColor(this.value),
            currentValue: this.value,
            dragging: false,
            visible: false,
            recommendedColor: [
                '#2d8cf0',
                '#19be6b',
                '#ff9900',
                '#ed4014',
                '#00b5ff',
                '#19c919',
                '#f9e31c',
                '#ea1a1a',
                '#9b1dea',
                '#00c2b1',
                '#ac7a33',
                '#1d35ea',
                '#8bc34a',
                '#f16b62',
                '#ea4ca3',
                '#0d94aa',
                '#febd79',
                '#5d4037',
                '#00bcd4',
                '#f06292',
                '#cddc39',
                '#607d8b',
                '#000000',
                '#ffffff'
            ]
        }
    },

    computed: {
        arrowClasses () {
            return [
                `${this.inputPrefixCls}-icon`,
                `${this.inputPrefixCls}-icon-normal`
            ]
        },
        transition () {
            return oneOf(this.placement, ['bottom-start', 'bottom', 'bottom-end']) ? 'slide-up' : 'fade'
        },
        saturationColors: {
            get () {
                return this.val
            },
            set (newVal) {
                this.val = newVal
                this.$emit('on-active-change', this.formatColor)
            }
        },
        classes () {
            return [
                `${this.prefixCls}`,
                {
                    [`${this.prefixCls}-transfer`]: this.transfer
                }
            ]
        },
        wrapClasses () {
            return [
                `${this.prefixCls}-rel`,
                `${this.prefixCls}-${this.size}`,
                `${this.inputPrefixCls}-wrapper`,
                `${this.inputPrefixCls}-wrapper-${this.size}`,
                {
                    [`${this.prefixCls}-disabled`]: this.disabled
                }
            ]
        },
        inputClasses () {
            return [
                `${this.prefixCls}-input`,
                `${this.inputPrefixCls}`,
                `${this.inputPrefixCls}-${this.size}`,
                {
                    [`${this.prefixCls}-focused`]: this.visible,
                    [`${this.prefixCls}-disabled`]: this.disabled
                }
            ]
        },
        dropClasses () {
            return [
                `${this.transferPrefixCls}-no-max-height`,
                {
                    [`${this.prefixCls}-transfer`]: this.transfer,
                    [`${this.prefixCls}-hide-drop`]: this.hideDropDown
                }
            ]
        },
        displayedColorStyle () {
            return {backgroundColor: toRGBAString(this.visible ? this.saturationColors.rgba : tinycolor(this.value).toRgb())}
        },
        formatColor () {
            const {format, saturationColors} = this

            if (format) {
                if (format === 'hsl') {
                    return tinycolor(saturationColors.hsl).toHslString()
                }

                if (format === 'hsv') {
                    return tinycolor(saturationColors.hsv).toHsvString()
                }

                if (format === 'hex') {
                    return saturationColors.hex
                }

                if (format === 'rgb') {
                    return toRGBAString(saturationColors.rgba)
                }
            } else if (this.alpha) {
                return toRGBAString(saturationColors.rgba)
            }

            return saturationColors.hex
        },
        confirmColorClasses () {
            return [
                `${this.prefixCls}-confirm-color`,
                {
                    [`${this.prefixCls}-confirm-color-editable`]: this.editable
                }
            ]
        },
        // 3.4.0, global setting customArrow ????????????arrow ?????????
        arrowType () {
            let type = 'xia'
            return type
        },
        // 3.4.0, global setting
        arrowSize () {
            let size = '12'
            return size
        }
    },

    watch: {
        value (newVal) {
            this.val = changeColor(newVal)
        },
        visible (val) {
            this.val = changeColor(this.value)
            this.$refs.drop[val ? 'update' : 'destroy']()
            this.$emit('on-open-change', Boolean(val))
        }
    },

    mounted () {
        this.$on('on-escape-keydown', this.closer)
        this.$on('on-dragging', this.setDragging)
    },

    methods: {
        setDragging (value) {
            this.dragging = value
        },
        handleClose (event) {
            if (this.visible) {
                if (this.dragging || event.type === 'mousedown') {
                    event.preventDefault()
                    return
                }

                if (this.transfer) {
                    const {$el} = this.$refs.drop
                    if ($el === event.target || $el.contains(event.target)) {
                        return
                    }
                }

                this.closer(event)
                return
            }

            this.visible = false
        },
        toggleVisible () {
            if (this.disabled) {
                return
            }

            this.visible = !this.visible
            this.$refs.input.focus()
        },
        childChange (data) {
            this.colorChange(data)
        },
        colorChange (data, oldHue) {
            this.oldHue = this.saturationColors.hsl.h
            this.saturationColors = changeColor(data, oldHue || this.oldHue)
        },
        closer (event) {
            if (event) {
                event.preventDefault()
                event.stopPropagation()
            }

            this.visible = false
            this.$refs.input.focus()
        },
        handleButtons (event, value) {
            this.currentValue = value
            this.$emit('input', value)
            this.$emit('on-change', value)
            this.dispatch('FormItem', 'on-form-change', value)
            this.closer(event)
        },
        handleSuccess (event) {
            this.handleButtons(event, this.formatColor)
            this.$emit('on-pick-success')
        },
        handleClear (event) {
            this.handleButtons(event, '')
            this.$emit('on-pick-clear')
        },
        handleSelectColor (color) {
            this.val = changeColor(color)
            this.$emit('on-active-change', this.formatColor)
        },
        handleEditColor (event) {
            const value = event.target.value
            this.handleSelectColor(value)
        },
        handleFirstTab (event) {
            if (event.shiftKey) {
                event.preventDefault()
                event.stopPropagation()
                this.$refs.ok.$el.focus()
            }
        },
        handleLastTab (event) {
            if (!event.shiftKey) {
                event.preventDefault()
                event.stopPropagation()
                this.$refs.saturation.$el.focus()
            }
        },
        onTab (event) {
            if (this.visible) {
                event.preventDefault()
            }
        },
        onEscape (event) {
            if (this.visible) {
                this.closer(event)
            }
        },
        onArrow (event) {
            if (!this.visible) {
                event.preventDefault()
                event.stopPropagation()
                this.visible = true
            }
        }
    }
}
</script>
