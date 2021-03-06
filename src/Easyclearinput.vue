<template>
    <div class="vc-easyclearinput-component form-group" :class="[statusClass, { 'has-feedback': icon }]" :style="{ 'width': optionalWidth }">
        <label class="label-item">{{ label }}</label>
        <div class="info-text" :class="infoTextClass">{{ infoText }}</div>
        <div :class="{ 'input-box': true, 'input-group': (slotBefore || slotAfter)}">
            <slot name="input-before"></slot>
            <span v-if="!slot-before || !slot-after" style="width: 1%;display: table-cell">&nbsp;</span><!-- 占位元素，用于撑开宽度，原因未知 -->
            <span v-if="icon" class="glyphicon form-control-feedback" :class="iconClass" aria-hidden="true"></span>
            <span @click="handleClear" class="clear-it glyphicon glyphicon-remove-circle" :class="{ 'has-icon': icon, 'hide': hideClearIcon, 'slot-after': slotAfter }" aria-hidden="true"></span>
            <input class="form-control"
                :class="[ 'form-control', slotBefore ? 'slot-before' : '', slotAfter ? 'slot-after' : '' ]"
                v-if="type !== 'textarea'"
                v-el:input
                :type="type"
                :disabled="disabled"
                :readonly="readonly"
                v-model="value"
                :placeholder="placeholder"
                @focus="handleFocus"
                @blur="handleBlur"
            />
            <textarea 
                v-if="type === 'textarea'"
                class="form-control"
                v-el:input
                :type="type"
                :disabled="disabled"
                :readonly="readonly"
                v-model="value"
                :placeholder="placeholder"
                @focus="handleFocus"
                @blur="handleBlur"
            >
            </textarea>
            <slot></slot>
            <slot name="input-after"></slot>
        </div>
    </div>
</template>

<style lang="less">
// container
.vc-easyclearinput-component {

    .label-item {
        font-weight: normal;
        display: table;
        vertical-align: bottom;
        float: left;
        height: 34px;
        line-height: 34px;
    }

    textarea.form-control {
        resize: vertical;
    }

    .glyphicon {
        z-index: 9;
    }

    .input-box {
        display: table;
        position: relative;

        .form-control {
            width: 100%;
            border-radius: 4px!important; 
            &.slot-before {
                border-top-left-radius: 0!important;
                border-bottom-left-radius: 0!important;
            }
            &.slot-after {
                border-top-right-radius: 0!important;
                border-bottom-right-radius: 0!important;
            }
        }
        &:hover {
            .clear-it {
                visibility: visible;
            }
        }

        .clear-it {
            visibility: hidden;
            position: absolute;
            top: 50%;
            right: 6px;
            // css3 it more flexible
            -webkit-transform: translateY(-50%);
            transform: translateY(-50%);
            opacity: .3;

            &.slot-after {

            }

            &.has-icon {
                right: 28px;
            }
        }
    }

    @success: #87d068;
    @warning: #fa0;
    @error: #f50;
    .info-text {
        position: absolute;
        top: -22px;

        &.with-success {
            color: @success;
        }
        &.with-warning {
            color: @warning;
        }
        &.with-error {
            color: @error;
        }
    }

} // end of container
</style>

<script>
const EVENT_DELAY = 128

export default {
    name: 'vc-easyclearinput',
    props: {
        type: {
            type: String,
            default: 'text'
        },
        value: {
            twoWay: true,
        },
        label: String,
        placeholder: String,
        infoText: {
            type: String,
            default: ''
        },
        disabled: {
            type: Boolean,
            default: false
        },
        readonly: {
            type: Boolean,
            default: false
        },
        autofocus: {
            type: Boolean,
            default: false
        },
        width: {
            type: [Number, String],
            default: '250' 
        },
        icon: {
            type: Boolean,
            default: false
        },
        status: {
            type: String,
            default: ''
        },
        onFocus: { // focus回调
            type: Function,
            default: function () {}
        },
        onBlur: { // blur回调
            type: Function,
            default: function () {}
        },
        onClear: { // onClear回调
            type: Function,
            default: function () {}
        }
    },
    data () {
        return {
            isClear: false,
            slotBefore: false,
            slotAfter: false
        }
    },
    created () {

    },
    ready () {
        if (this.autofocus) {
            this.focusInput()
        }
        // 检查是否有用户自定义slot传入(input-before)
        this.checkSlot()
        // 检查用户是否内联了不该内联的事件(focus & blur)
        this.checkEvents()
    },
    computed: {
        hideClearIcon () {
            return (this.value == null) || (this.value === '') || this.disabled || this.readonly
        },
        optionalWidth () {
            if (this.width == null || this.width === '') {
                return null
            }
            if (Number.isInteger(+this.width)) {
                return this.width + 'px'
            }
            return this.width
        },
        statusClass () {
            return 'has-' + this.status
        },
        infoTextClass () {
            return 'with-' + this.status
        },
        iconClass () {
            if (this.status === 'success') {
                return 'glyphicon-ok'
            }
            if (this.status === 'warning') {
                return 'glyphicon-warning-sign'
            }
            if (this.status === 'error') {
                return 'glyphicon-remove'
            }
        }
    },
    watch: {
        autofocus (val) {
            if (val) {
                this.focusInput()
            }
        }
    },
    methods: {
        checkSlot () {
            var keys = Object.keys(this._slotContents)
            for (let i = 0, len = keys.length; i < len; i++) {
                if (keys[i] === 'input-before') {
                    this.slotBefore = true
                }
                if (keys[i] === 'input-after') {
                    this.slotAfter = true
                }
            }
        },
        checkEvents () {
            let focus = this._events.focus
            if (focus && focus.length > 0) {
                console.warn('if you want to listen on focus event, please use `:on-focus` callback!')
            }
            let blur = this._events.blur
            if (blur && blur.length > 0) {
                console.warn('if you want to listen on blur event, please use `:on-blur` callback!')
            }
        },
        /**
         * 点击清除按钮
         * 1. blur 2. clear 3.focus again
         * 事件修正，使得小叉号成为类似系统原生的和input一体的控件，
         * 点击小叉号不应该带来input的失焦还有相应事件的响应
         */
        focusInput () { // 工具方法
            this.$els.input && this.$els.input.focus()
        },
        handleBlur (e) {
            // console.log(1)
            setTimeout(() => {
                if (!this.isClear) {
                    this.onBlur(e)
                } else {
                    setTimeout(() => {
                        this.focusInput()
                    }, 0)
                }
                // this.isClear = false
            }, EVENT_DELAY)
        },
        handleClear () {
            // console.log(2)
            // 可编辑状态下
            if (!this.disabled && !this.readonly) {
                this.isClear = true
                this.value = ''
                this.onClear()
                this.focusInput()
            }
        },
        handleFocus (e) {
            // console.log(3)
            setTimeout(() => {
                if (!this.isClear) {
                    this.onFocus(e)
                }
                this.isClear = false
            }, EVENT_DELAY + 10)
        }
    }
}
</script>
