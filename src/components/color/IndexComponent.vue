<template>
    <div
        class="color-picker"
        :class="{ light: isLightTheme }"
    >
        <div class="mode-chooser">
            <div class="chooser-text" @click="modeChange" :class="{'selected': customMode}">基础模式</div>
            <div class="chooser-text" @click="modeChange" :class="{'selected': !customMode}">高级模式</div>
        </div>
        <div class="color-set"  v-if="!customMode">
            <SaturationComponent
                ref="saturation"
                :color="rgbString"
                :hsv="hsv"
                :size="hueHeight"
                @selectSaturation="selectSaturation"
            />
            <HueComponent
                ref="hue"
                :hsv="hsv"
                :width="hueWidth"
                :height="hueHeight"
                @selectHue="selectHue"
            />
            <AlphaComponent
                ref="alpha"
                :color="rgbString"
                :rgba="rgba"
                :width="hueWidth"
                :height="hueHeight"
                @selectAlpha="selectAlpha"
            />
        </div>
        <div
            :style="{ height: previewHeight + 'px' }"
            class="color-show"
        >
            <PreviewComponent
                :color="rgbaString"
                :width="previewWidth"
                :height="previewHeight"
            />
            <SuckerComponent
                v-if="!suckerHide"
                :sucker-canvas="suckerCanvas"
                :sucker-area="suckerArea"
                @openSucker="openSucker"
                @selectSucker="selectSucker"
            />
        </div>
        <BoxComponent
            name="HEX"
            :color="modelHex"
            @inputColor="inputHex"
            @blur="handleBlur"
        />
        <BoxComponent
            name="RGBA"
            :color="modelRgba"
            @inputColor="inputRgba"
            @blur="setColorsHistory(modelHex)"
        />
        <ColorsComponent
            v-if="customMode"
            :color="rgbaString"
            :colors-default="colorsDefault"
            :colors-history-key="colorsHistoryKey"
            @selectColor="selectColor"
        />
        <HistoryComponent :colors="[modelHex,...colorsHistory]"/>
        <!-- </div> -->
    </div>
</template>

<script>
import mixin from './mixin'
import SaturationComponent from './SaturationComponent.vue'
import HueComponent from './HueComponent.vue'
import AlphaComponent from './AlphaComponent.vue'
import PreviewComponent from './PreviewComponent.vue'
import SuckerComponent from './SuckerComponent.vue'
import BoxComponent from './BoxComponent.vue'
import ColorsComponent from './ColorsComponent.vue'
import HistoryComponent from './HistoryComponent.vue'
export default {
    components: {
        SaturationComponent,
        HueComponent,
        AlphaComponent,
        PreviewComponent,
        SuckerComponent,
        BoxComponent,
        ColorsComponent,
        HistoryComponent
    },
    mixins: [mixin],
    props: {
        color: {
            type: String,
            default: '#000000'
        },
        theme: {
            type: String,
            default: 'dark'
        },
        suckerHide: {
            type: Boolean,
            default: true
        },
        suckerCanvas: {
            type: null, // HTMLCanvasElement
            default: null
        },
        suckerArea: {
            type: Array,
            default: () => []
        },
        colorsDefault: {
            type: Array,
            default: () => [
                '#000000', '#FFFFFF', '#FF1900', '#F47365', '#FFB243', '#FFE623', '#6EFF2A', '#1BC7B1',
                '#00BEFF', '#2E81FF', '#5D61FF', '#FF89CF', '#FC3CAD', '#BF3DCE', '#8E00A7', 'rgba(0,0,0,0)'
            ]
        },
        colorsHistoryKey: {
            type: String,
            default: 'vue-colorpicker-history'
        }
    },
    data() {
        return {
            hueWidth: 15,
            hueHeight: 152,
            previewHeight: 30,
            modelRgba: '',
            modelHex: '',
            colorsHistory: [],
            customMode: true,
            chooserList:[{name:'基础面板',selected:true},{name:'高级面板',selected:false}],
            r: 0,
            g: 0,
            b: 0,
            a: 1,
            h: 0,
            s: 0,
            v: 0
        }
    },
    computed: {
        isLightTheme() {
            return this.theme === 'light'
        },
        totalWidth() {
            return this.hueHeight + (this.hueWidth + 8) * 2
        },
        previewWidth() {
            return this.totalWidth - (this.suckerHide ? 0 : this.previewHeight)
        },
        rgba() {
            return {
                r: this.r,
                g: this.g,
                b: this.b,
                a: this.a
            }
        },
        hsv() {
            return {
                h: this.h,
                s: this.s,
                v: this.v
            }
        },
        rgbString() {
            return `rgb(${this.r}, ${this.g}, ${this.b})`
        },
        rgbaStringShort() {
            return `${this.r}, ${this.g}, ${this.b}, ${this.a}`
        },
        rgbaString() {
            return `rgba(${this.rgbaStringShort})`
        },
        hexString() {
            return this.rgb2hex(this.rgba, true)
        }
    },
    created() {
        Object.assign(this, this.setColorValue(this.color))
        this.setText()

        // 避免初始化时，也会触发changeColor事件
        this.$watch('rgba', () => {
            this.$emit('changeColor', {
                rgba: this.rgba,
                hsv: this.hsv,
                hex: this.modelHex
            })
        })
    },
    methods: {
        modeChange(e) {
            // console.log(e.target.innerHTML);
            e.target.innerHTML == '基础模式'?this.customMode = true: this.customMode = false;
        },
        setColorsHistory(color) {
            // console.log(color);
            if (!color) {
                return
            }
            const colors = this.colorsHistory
            const index = colors.indexOf(color)
            if (index >= 0) {
                colors.splice(index, 1)
            }
            if (colors.length > 3) {
                colors.length = 3
            }
            colors.unshift(color)
            this.colorsHistory = colors
            localStorage.setItem(this.colorsHistoryKey, JSON.stringify(colors))
        },
        selectSaturation(color) {
            const { r, g, b, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, h, s, v })
            this.setText()
            // this.setColorsHistory(color)
        },
        selectHue(color) {
            const { r, g, b, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, h, s, v })
            this.setText()
            this.$nextTick(() => {
                this.$refs.saturation.renderColor()
                this.$refs.saturation.renderSlide()
            })
        },
        selectAlpha(a) {
            this.a = a
            this.setText()
        },
        inputHex(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, a, h, s, v })
            this.modelHex = color
            this.modelRgba = this.rgbaStringShort
            this.$nextTick(() => {
                if (!this.customMode) {
                    this.$refs.saturation.renderColor()
                    this.$refs.saturation.renderSlide()
                    this.$refs.hue.renderSlide()
                }
            })
            if (color.length == 7)
            this.setColorsHistory(color)
        },
        inputRgba(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, a, h, s, v })
            this.modelHex = this.hexString
            this.modelRgba = color
            // this.setColorsHistory(color)
            this.$nextTick(() => {
                if (!this.customMode) {
                    this.$refs.saturation.renderColor()
                    this.$refs.saturation.renderSlide()
                    this.$refs.hue.renderSlide()
                }
            })
        },
        setText() {
            this.setColorsHistory(this.modelHex)
            this.modelHex = this.hexString
            this.modelRgba = this.rgbaStringShort
        },
        openSucker(isOpen) {
            this.$emit('openSucker', isOpen)
        },
        selectSucker(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, a, h, s, v })
            this.setText()
            this.$nextTick(() => {
                if (!this.customMode) {
                    this.$refs.saturation.renderColor()
                    this.$refs.saturation.renderSlide()
                    this.$refs.hue.renderSlide()
                }
            })
        },
        selectColor(color) {
            const { r, g, b, a, h, s, v } = this.setColorValue(color)
            Object.assign(this, { r, g, b, a, h, s, v })
            this.setText()
            
        }
    }
}
</script>

<style lang="scss">
$color-width: 30px;
$top-height: 40px;
.color-picker {
    width: $color-width*8;
    padding: 20px;
    background: #1d2024;
    border-radius: 4px;
    box-shadow: 0 0 16px 0 rgba(0, 0, 0, 0.16);
    z-index: 1;
    .mode-chooser {
        display: flex;
        width: 100%;
        height: $top-height;
        border-bottom: 2px solid gray;
        margin-bottom: 10px;
        .selected {
            color: aqua;
        }
        .chooser-text {
            width: 50%;
            text-align: center;
            line-height: $top-height;
            cursor: pointer;
        }
    }
    &.light {
        background: #f7f8f9;
        .color-show {
            .sucker {
                background: #eceef0;
            }
        }
        .color-type {
            .name {
                background: #e7e8e9;
            }
            .value {
                color: #666;
                background: #eceef0;
            }
        }
        .colors.history {
            border-top: 1px solid #eee;
        }
    }
    canvas {
        vertical-align: top;
    }
    .color-set {
        display: flex;
    }
    .color-show {
        margin-top: 8px;
        display: flex;
    }
}
</style>
