
<template>
    <div class="eve-form">
        <el-form :ref="'formModel'" :label-width="labelWidth" :label-suffix="':'"
            :class="{ 'input-form': formData.inputForm, 'search-form': !formData.inputForm }" :model="formData.formModel"
            :rules="formData.rules" v-bind="$attrs" v-on="$listeners">
            <slot name="formTop" />
            <el-row :gutter="10">
                <el-col v-for="(item, key) in formContent" v-show="!item.hidden" :key="key"
                    :span="item.colSpan || formData.colSpan || 6">
                    <el-form-item :label="isLang ? $t(matchKey(item.placeholder || item.formItemLabel)) : item.formItemLabel"
                        :prop="item.Model">
                        <!-- 第一个自定义插槽 -->
                        <template v-if="item.formType == 'slot'">
                            <slot name="custom-slot" :data="item"></slot>
                        </template>
                        <!-- 第二个自定义插槽 -->
                        <template v-if="item.formType == 'slotTwo'">
                            <slot name="custom-slot-two" :data="item"></slot>
                        </template>
                        <!-- SWItch开关 -->
                        <el-switch v-if="item.formType == 'switch'" v-model="formData.formModel[item.Model]"
                            active-color="#4A7DFF" inactive-color="#dddddd">
                        </el-switch>

                        <!-- 输入框 -->
                        <el-input v-if="item.formType == 'text' || item.formType == 'textarea'" style="width: 100%"
                            v-model="formData.formModel[item.Model]" :type="item.formType" show-word-limit
                            :clearable="item.clearable" :disabled="item.disabled" :maxlength="item.maxlength"
                            :autosize="{ minRows: 3 }"
                            :placeholder="isLang ? $t(matchKey(item.placeholder || item.formItemLabel)) : item.placeholder || `请输入${item.formItemLabel}`"
                            :readonly="item.readonly" @keyup.enter.native="(e) => issueEvent(e, item.enterEvent)" />
                        <!-- 仅展示 -->
                        <el-input v-if="item.formType == 'viewText' || item.formType == 'viewTextarea'"
                            style="width: 100%" v-model="formData.formModel[item.Model]"
                            :type="item.formType == 'viewText' ? 'text' : 'textarea'" show-word-limit
                            :maxlength="item.maxlength" :autosize="{ minRows: 3 }" :readonly="item.readonly"
                            @keyup.enter.native="(e) => issueEvent(e, item.enterEvent)" />
                        <!--数字选择器-->
                        <el-input-number v-if="item.formType == 'text-number'" style="width: 100%"
                            v-model="formData.formModel[item.Model]" :min="item.min" :max="item.max"
                            :disabled="item.disabled" :maxlength="item.maxlength" :autosize="{ minRows: 3 }"
                            :placeholder="isLang ? $t(matchKey(item.placeholder || item.formItemLabel)) : item.placeholder || `请输入${item.formItemLabel}`"
                            :readonly="item.readonly" @keyup.enter.native="(e) => issueEvent(e, item.enterEvent)" />
                        <el-autocomplete v-if="item.formType == 'autocomplete'" v-model="formData.formModel[item.Model]"
                            :placeholder="isLang ? $t(matchKey(item.placeholder || item.formItemLabel)) : item.placeholder || `请输入${item.formItemLabel}`"
                            clearable :disabled="item.disabled"
                            :fetch-suggestions="(e) => issueEvent(e, item.fetchSuggestions)" style="width: 100%;" />
                        <!-- 选择器 -->
                        <el-select v-if="item.formType == 'select'" style="width: 100%"
                            v-model="formData.formModel[item.Model]"
                            :placeholder="isLang ? $t(matchKey(item.placeholder || item.formItemLabel)) : item.placeholder || `请输入${item.formItemLabel}`"
                            clearable filterable :multiple="item.multiple" :collapse-tags="true"
                            :disabled="item.disabled" :remote="item.remote"
                            :remote-method="(e) => issueEvent(e, item.remoteMethod)"
                            @change="(e) => issueEvent(e, item.onChange)" @focus="(e) => issueEvent(e, item.onFocus)">
                            <el-option v-for="i in dicts[item.dict] || item.options" :key="i.value" :label="i.label"
                                :value="item.isNum ? parseInt(i.value) : i.value"
                                :disabled="item.itemdisabled" />
                        </el-select>
                        <!-- 单选框 -->
                        <el-radio-group v-if="item.formType == 'radio' || item.formType == 'radioButton'"
                            v-model="formData.formModel[item.Model]" :size="item.size">
                            <!-- 普通单选框 -->
                            <template v-if="item.formType == 'radio'">
                                <el-radio v-for="radio in dicts[item.dict] || item.radios"
                                    :key="item.isNum ? parseInt(radio.value) : radio.value"
                                    :label="item.isNum ? parseInt(radio.value) : radio.value" :border="radio.border"
                                    :disabled="radio.disabled">{{ radio.label }}</el-radio>
                            </template>
                            <!-- 按钮形式的单选框 -->
                            <template v-if="item.formType == 'radioButton'">
                                <el-radio-button v-for="radio in dicts[item.dict] || item.radios"
                                    :key="item.isNum ? parseInt(radio.value) : radio.value"
                                    :label="item.isNum ? parseInt(radio.value) : radio.value"
                                    :disabled="radio.disabled">{{ radio.label }}</el-radio-button>
                            </template>
                        </el-radio-group>
                        <!-- 复选框 -->
                        <el-checkbox-group v-if="item.formType == 'checkboxgruop'"
                            v-model="formData.formModel[item.Model]" :size="item.size">
                            <el-checkbox v-for="(checkbox, key2) in item.checkbox" :key="key2" :label="checkbox.label"
                                :border="checkbox.border" :disabled="checkbox.disabled" :name="item.Model">
                                {{ checkbox.label }}
                            </el-checkbox>
                        </el-checkbox-group>
                        <!-- 日期时间选择器 -->
                        <el-date-picker style="width: 100%" v-if="item.formType == 'dateTimePicker'"
                            v-model="formData.formModel[item.Model]" :disabled="item.disabled"
                            :type="item.type || 'daterange'"
                            :placeholder="isLang ? $t(matchKey(item.placeholder)) : item.placeholder"
                            :clearable="item.clearable === false ? false : true"
                            :picker-options="item.pickerOptions"
                            :start-placeholder="item.startPlaceholder ? $t(item.startPlaceholder || 'system.start_date') : '开始日期'"
                            :end-placeholder="item.endPlaceholder ? $t(item.endPlaceholder || 'system.end_date') : '结束日期'"
                            :format="item.format || 'yyyy-MM-dd'" :value-format="item.valueFormat || 'yyyy-MM-dd'" />
                              <!--  年份月份 -->
                        <el-date-picker
                            style="width: 100%"
                            v-if="item.formType == 'month' || item.formType == 'year'"
                            v-model="formData.formModel[item.Model]"
                            :disabled="item.disabled"
                            :type="item.formType"
                            :placeholder="isLang?$t(matchKey(item.placeholder)):item.placeholder"
                            :clearable="item.clearable === false? false: true"
                            :start-placeholder="item.startPlaceholder ? $t(item.startPlaceholder || 'system.start_date') : '开始日期'"
                            :end-placeholder="item.endPlaceholder ? $t(item.endPlaceholder || 'system.end_date') : '结束日期'"
                            :value-format="item.valueFormat || 'yyyy-MM-dd'"
                        />

                        <el-cascader v-if="item.formType == 'cascader'" style="width: 100%"
                            v-model="formData.formModel[item.Model]" :options="item.options" />
                        <slot :name="item.slot"></slot>
                    </el-form-item>
                </el-col>
                <el-col :span="actionSpan" v-if="!formData.hideFormAction" style="text-align: right;">
                    <el-form-item label-width="auto">
                        <slot name="formActionBefore"> </slot>
                        <slot name="formAction" v-if="isLang">
                            <el-button v-if="formData.showFormCancel" @click="eveFormCancel()">{{ $t(formData.cancelText
            ||
            'system.cancel') }}</el-button>
                            <el-button type="primary" @click="formSubmit()"> {{ $t(formData.submitText ||
            'system.query') }}</el-button>
                            <el-button v-if="!formData.showFormReset" @click="eveFormReset()">{{ $t('system.reset')
                                }}</el-button>
                        </slot>
                        <slot name="formAction" v-else>
                            <el-button v-if="formData.showFormCancel" @click="eveFormCancel()">{{ formData.cancelText ||
            '取消' }}</el-button>
                            <el-button type="primary" @click="formSubmit()">{{ formData.submitText || '查询' }}</el-button>
                            <el-button v-if="!formData.showFormReset" @click="eveFormReset()">{{ '重置' }}</el-button>
                        </slot>
                        <slot name="formAdvancedButton">
                            <!-- 不显示收起按钮 -->
                            <el-button v-if="!formData.hideFormAdvanced" type="text"
                                @click="toggleAction">{{advanceText || $t('system_unfold') }} <svg-icon icon-class="advanced"
                                    class="advance-icon"
                                    :style="{transform: isAdvanced ? 'rotate(0deg)' : 'rotate(180deg)', transition: 'transform .5s ease'}"></svg-icon></el-button>
                        </slot>
                        <slot name="formActionEnd"> </slot>
                    </el-form-item>
                </el-col>
            </el-row>
        </el-form>
    </div>
</template>

<script>

export default {
    props: {
        formData: {
            type: Object,
            default: () => { }
        },
        isLang: {
            type: Boolean,
            default: false
        },
        matchKey: {
            type: Function,
            default: (name) => { }
        },
        formToggle: {
            type: Function,
            default: () => { }
        },
        labelWidth: {
            type: String,
            default: 'auto'
        }
    },

    computed: {
        actionSpan() {
            // 根据展示内容判断按钮占多少列
            const formList = this.formContent?.filter(el => {
                return !el.hidden;
            }) ?? []
            const spanTotal = formList.map(el => el.colSpan || this.formData.colSpan).reduce((cur, prev) => {
                return cur + prev
            }, 0)
            return 24 - spanTotal % 24;
        },
        dicts() {
            return { ...this.dictlist }
        }
    },
    watch: {
        formData: {
            handler(value, oldVal) {
                this.formContent = value.formContent
                if (!this.formData.colSpan) {
                    this.formData.colSpan = 6
                }
            },
            deep: true,
            immediate: true
        },
        isAdvanced: {
            handler(value) {
                if (value) {
                    const formContent = this.formContent.map((el, i) => {
                        el['hidden'] = false;
                        return el;
                    })
                    this.$set(this, 'formContent', formContent)
                } else {
                    // 收起时判断去除按钮占多少列，优先取默认按钮列
                    let span = 24 - (this.formData.actionSpan || this.formData.colSpan);
                    span = span > 0 ? span : 24
                    const len = (span / this.formData.colSpan) || 1;
                    const formContent = this.formContent.map((el, i) => {
                        if (i < len) {
                            el['hidden'] = false;
                        } else {
                            el['hidden'] = true;
                        }
                        return el;
                    })
                    this.$set(this, 'formContent', formContent)
                }
            },
            immediate: true
        }
    },
    data() {
        return {
            formContent: [],
            isAdvanced: false,
            advanceText: '',
            formModel: { ...this.formData.formModel },
            dictlist: {}
        }
    },
    created() {
        // 处理字典数据问题
        const dicts_list = [...new Set(this.formData.dicts || this.formData.formContent.filter(el => el.dict).map(el => el.dict))] || []
        const dicts_pro = dicts_list.map(el => this.getDicts(el)) || []
        const that = this;
        Promise.all(dicts_pro).then(res => {
            res.map((el, i) => {
                const { data } = el
                const key = data[0].dictType
                const value = data.map(e => {
                    return {
                        label: e.dictLabel,
                        value: e.dictValue
                    }
                }) || []
                that.$set(that.dictlist, key, value)
            })
        })
    },
    mounted() {
        this.isAdvanced = !!this.formData.isAdvanced;
    },
    methods: {
        toggleAction() {
            this.advanceText = this.isAdvanced ? this.$t('system_unfold') : this.$t('system_fold');
            this.isAdvanced = !this.isAdvanced
            this.$props?.formToggle && this.$props?.formToggle()
        },
        issueEvent(value, event) {
            if (event) {
                return event(value);
            }
        },
        // 重置Form
        eveFormReset() {
            this.$refs["formModel"].resetFields()
            this.$emit('eveFormSubmit', { ...this.formData.formModel });
        },
        // 取消Form
        eveFormCancel() {
            this.$refs["formModel"].resetFields()
            this.$emit('eveFormCancel', { ...this.formData.formModel });
        },
        // 提交Form
        formSubmit() {
            this.$refs["formModel"].validate(valid => {
                if (valid) {
                    this.$emit('eveFormSubmit', this.formData.formModel)
                }
            })
        },
        resetFields() {
            this.$refs["formModel"].resetFields()
        }
    }
}
</script>

<style lang="scss">
.eve-form {
    .el-form-item--mini .el-form-item__label {
        font-size: 12px !important;
        padding-right: 8px !important;
    }

    .el-button--mini {
        font-size: 12px !important;
        padding: 2px 8px !important;
        line-height: 20px !important;
    }

    .el-button--text {
        color: rgba(0, 0, 0, 0.45);
        &:focus {
            color: rgba(0, 0, 0, 0.45);
        }
        &:hover {
            color: #4A7DFF;
        }
    }
    .vue-treeselect__control{
        height: 30px;
    }
    .vue-treeselect__placeholder, .vue-treeselect__single-value{
        line-height: 30px;
        font-size: 12px;
    }
    .el-form-item--mini .el-form-item__content{
        min-height: 30px;
    }
    .input-form{
        .el-form-item--mini.el-form-item{
            margin-bottom: 20px !important;
        }
    }
    .search-form{
        .el-form-item--mini.el-form-item{
            margin-bottom: 10px !important;
        }
    }
}
</style>
