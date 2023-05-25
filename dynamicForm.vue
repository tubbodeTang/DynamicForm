<template>
  <div class="form-row" :class="{ 'show-form': !isEdit }">
    <div
      class="form-item-col"
      :class="{ 'single-col': formItem.single }"
      v-for="formItem in formInfo"
      :key="formItem.key"
    >
      <template v-if="isEdit">
        <el-form-item
          v-if="formType == 'object'"
          :label-width="labelWidth"
          :label="formItem.name"
          :prop="formKey + '.' + formItem.key"
          :rules="rules[formItem.key]"
        >
          <template v-if="formItem.type.indexOf('input') > -1">
            <el-input
              v-model="formModule[formKey][formItem.key]"
              :type="formItem.type == 'inputArea' ? 'textarea' : ''"
              :placeholder="!formItem.unit ? '请输入' + formItem.name : ''"
              :rows="3"
              ><i
                slot="suffix"
                style="font-style: normal; margin-right: 10px"
                >{{ formItem.unit }}</i
              ></el-input
            >
          </template>
          <template v-if="formItem.type == 'additionalSelect'">
            <addition-select
              :listData="formOptions[formItem.options]"
              :model="formModule[formKey][formItem.key]"
              :isMultiple="formItem.multiple"
              :keyName="formItem.optionVal"
              :labelName="formItem.optionLabel"
              :additionName="'新增'"
              :additionFunc="showAddNewDialog"
              :additionParam="formKey + '.' + formItem.key"
              @selectChange="
                ($event) => {
                  setValue(formKey, formItem.key, $event);
                }
              "
            ></addition-select>
          </template>
          <template v-if="formItem.type == 'select'">
            <el-select
              v-model="formModule[formKey][formItem.key]"
              :filterable="formItem.filterable"
              :multiple="formItem.multiple"
              :allow-create="formItem.allowCreate"
              :default-first-option="formItem.filterable"
              clearable
            >
              <i slot="suffix" style="font-style: normal; margin-right: 10px">{{
                formItem.unit
              }}</i>
              <template v-for="i in formOptions[formItem.options]">
                <el-option
                  :label="i[formItem.optionLabel]"
                  :value="i[formItem.optionVal]"
                  :key="i[formItem.optionVal]"
                ></el-option>
              </template>
            </el-select>
          </template>
          <template v-if="formItem.type == 'dateRange'">
            <el-date-picker
              v-model="formModule[formKey][formItem.key]"
              clearable
              unlink-panels
              type="daterange"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 100%"
            >
            </el-date-picker>
          </template>
        </el-form-item>
        <el-form-item
          v-else
          :label-width="labelWidth"
          :label="formItem.name"
          :prop="formKey + '.' + formIndex + '.' + formItem.key"
          :rules="rules[formItem.key]"
        >
          <template v-if="formItem.type.indexOf('input') > -1">
            <el-input
              v-model="formModule[formKey][formIndex][formItem.key]"
              :type="formItem.type == 'inputArea' ? 'textarea' : ''"
              :rows="3"
              ><i
                slot="suffix"
                style="font-style: normal; margin-right: 10px"
                >{{ formItem.unit }}</i
              ></el-input
            >
          </template>
          <template v-if="formItem.type == 'additionalSelect'">
            <addition-select
              :listData="formOptions[formItem.options]"
              :model="formModule[formKey][formIndex][formItem.key]"
              :isMultiple="formItem.multiple"
              :keyName="formItem.optionVal"
              :labelName="formItem.optionLabel"
              :additionName="'新增'"
              :additionFunc="showAddNewDialog"
              :additionParam="formKey + '.' + formItem.key"
              @selectChange="
                ($event) => {
                  setValue(formKey, formItem.key, $event);
                }
              "
            ></addition-select>
          </template>
          <template v-if="formItem.type == 'select'">
            <el-select
              v-model="formModule[formKey][formIndex][formItem.key]"
              :filterable="formItem.filterable"
              :multiple="formItem.multiple"
              :allow-create="formItem.allowCreate"
              :default-first-option="formItem.filterable"
              clearable
            >
              <i slot="suffix" style="font-style: normal; margin-right: 10px">{{
                formItem.unit
              }}</i>
              <template v-for="i in formOptions[formItem.options]">
                <el-option
                  :label="i[formItem.optionLabel]"
                  :value="i[formItem.optionVal]"
                  :key="i[formItem.optionVal]"
                ></el-option>
              </template>
            </el-select>
          </template>
          <template v-if="formItem.type == 'dateRange'">
            <el-date-picker
              v-model="formModule[formKey][formIndex][formItem.key]"
              clearable
              unlink-panels
              type="daterange"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 100%"
            >
            </el-date-picker>
          </template>
        </el-form-item>
      </template>
      <template v-else>
        <!-- 非编辑状态 -->
        <div class="show-info-item">
          <span class="left">{{ formItem.name + ':' }}</span>
          <span class="right"
            ><template
              v-if="
                formItem.type == 'input' ||
                formItem.type == 'inputArea' ||
                formItem.type == 'dateRange'
              "
              >{{
                formType == 'object'
                  ? formModule[formKey][formItem.key] || '--'
                  : formModule[formKey][formIndex][formItem.key] || '--'
              }}</template
            >
            <template
              v-if="
                formItem.type == 'select' || formItem.type == 'additionalSelect'
              "
            >
              {{
                formType == 'object'
                  ? formModule[formKey][formItem.key + 'Name'] ||
                    formModule[formKey][formItem.key] ||
                    '--'
                  : formModule[formKey][formIndex][formItem.key + 'Name'] ||
                    formModule[formKey][formIndex][formItem.key] ||
                    '--'
              }}</template
            >
            <span class="unit">{{ formItem.unit }}</span></span
          >
        </div>
      </template>
    </div>
    <!-- 奇数项显示补齐 -->
    <div v-if="!isEdit && showSlot" class="form-item-col">
      <div class="show-info-item">
        <span class="left"></span>
        <span class="right"></span>
      </div>
    </div>
    <!-- 额外行-缺失说明 -->
    <div
      class="appendix show-info-item"
      v-if="!isEdit && formModule[commentKey]"
    >
      <span class="left">
        <el-tooltip placement="top" class="no-print">
          <div slot="content">
            如无法提供字段信息，请如实填写每个空格的缺失原因
          </div>
          <i class="el-icon-question"></i
        ></el-tooltip>
        <span class="comment">缺失说明</span>
      </span>
      <span class="right">{{ formModule[commentKey] }}</span>
    </div>
  </div>
</template>

<script>
import AdditionSelect from '@/components/common/additionSelect.vue';
export default {
  components: {
    AdditionSelect,
  },
  props: {
    labelWidth: {
      // 标签宽度
      type: String,
      default: '',
    },
    isEdit: {
      // 是否能编辑
      type: Boolean,
      default: false,
    },
    // 表单字段数据
    formInfo: {
      type: Array,
      default: function () {
        return [];
      },
    },
    // 表单标识
    formKey: {
      type: String,
      default: '',
    },
    // 说明标志
    commentKey: {
      type: String,
      default: '',
    },
    // 表单值对象
    formModule: {
      type: Object,
      default: function () {
        return {};
      },
    },
    formType: {
      type: String,
      default: 'object',
    },
    formIndex: {
      type: Number,
      default: 0,
    },
    // 表单下拉数据
    formOptions: {
      type: Object,
      default: function () {
        return {};
      },
    },
    // 表单校验规则
    rules: {
      type: Object,
      default: function () {
        return {};
      },
    },
  },
  data() {
    return {};
  },
  computed: {
    showSlot() {
      // 是否显示奇数项补齐
      return (
        (this.formInfo.length -
          this.formInfo.filter((item) => item.single).length) %
          2 !=
        0
      );
    },
  },
  methods: {
    // 新增下拉项目
    showAddNewDialog(formKey) {
      this.$emit('addNewSelectItem', formKey);
    },
    setValue(formKey, formItemKey, val) {
      this.$set(this.formModule[formKey], formItemKey, val);
    },
  },
};
</script>

<style scoped></style>
