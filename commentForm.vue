<template>
  <div class="form-row">
    <div
      class="form-item-col"
      :span="12"
      v-for="formItem in formInfo"
      :key="formItem.key"
    >
      <template v-if="isEdit">
        <el-form-item
          :label="formItem.name"
          :prop="formKey + '.' + formItem.key"
          :style="{ width: formItem.showCommet ? '47%' : '100%' }"
        >
          <template v-if="formItem.type == 'input'">
            <el-input
              v-model="formModule[formKey][formItem.key]"
              :style="{ width: formItem.showCommet ? '100%' : '76%' }"
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
              :style="{ width: formItem.showCommet ? '100%' : '76%' }"
            ></addition-select>
          </template>
          <template v-if="formItem.type == 'select'">
            <el-select
              v-model="formModule[formKey][formItem.key]"
              :filterable="formItem.filterable"
              :allow-create="formItem.allowCreate"
              :default-first-option="formItem.filterable"
              clearable
              :style="{ width: formItem.showCommet ? '100%' : '76%' }"
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
              :style="{ width: formItem.showCommet ? '100%' : '76%' }"
            >
            </el-date-picker>
          </template>
          <span
            class="commet-btn"
            v-if="!formItem.showCommet"
            :style="{
              color: formItem.invalid ? '#409eff' : '#ccc',
              cursor: formItem.invalid ? 'pointer' : 'not-allowed',
            }"
            @click="formItem.showCommet = formItem.invalid && true"
            >缺失说明</span
          >
        </el-form-item>
        <div class="reason" v-if="formItem.showCommet" style="width: 53%">
          <el-form-item :prop="formKey + '.' + formItem.key + 'Comment'">
            <el-input
              :placeholder="'缺失说明'"
              type="textarea"
              autosize
              v-model="formModule[formKey][formItem.key + 'Comment']"
              style="width: 90%"
            ></el-input>
            <el-button
              type="text"
              v-if="formItem.showCommet"
              @click="formItem.showCommet = false"
              >╳</el-button
            >
          </el-form-item>
        </div>
      </template>
      <template v-else>
        <!-- 非编辑状态 -->
        <el-form-item
          class="show-item"
          :label="formItem.name"
          style="width: 100%"
        >
          <template
            v-if="
              !formModule[formKey][formItem.key] &&
              !formModule[formKey][formItem.key + 'Name']
            "
          >
            <el-tooltip placement="top" class="no-print">
              <div slot="content">
                <span>{{ formModule[formKey][formItem.key + 'Comment'] }}</span>
              </div>
              <span class="comment"><i class="el-icon-info"></i>缺失说明</span>
            </el-tooltip>
            <span class="comment-print">
              <span class="comment"
                >（缺失说明：{{
                  formModule[formKey][formItem.key + 'Comment']
                }})</span
              >
            </span>
          </template>
          <template v-else>
            <template v-if="formItem.type == 'input'">{{
              formModule[formKey][formItem.key] || '--'
            }}</template>
            <template
              v-if="
                formItem.type == 'select' || formItem.type == 'additionalSelect'
              "
            >
              {{
                formModule[formKey][formItem.key + 'Name'] ||
                formModule[formKey][formItem.key] ||
                '--'
              }}</template
            >
            <span class="unit">{{ formItem.unit }}</span>
          </template>
        </el-form-item>
      </template>
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
    // 表单值对象
    formModule: {
      type: Object,
      default: function () {
        return {};
      },
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
  computed: {},
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

<style scoped>
.form-row {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
}
.form-item-col {
  display: flex;
  /* flex-direction: column; */
  width: 50%;
}
.form-item-col >>> .el-form-item__label {
  line-height: 18px !important;
}
.form-item-col .reason >>> .el-form-item__content {
  margin-left: 10px !important;
}
.form-item-col .commet-btn {
  margin-left: 10px;
  padding-bottom: 16px;
  color: #409eff;
  font-size: 12px;
}
.form-item-col .commet-btn:hover {
  cursor: pointer;
}
>>> .show-item .el-form-item__content {
  color: #999;
  margin-left: 10px;
}
>>> .show-item .el-form-item__label {
  max-width: 180px;
  min-width: 180px;
}
.comment {
  font-size: 14px;
  color: #409eff;
  position: relative;
  right: 0;
  display: inline-block;
}
.comment-print {
  display: none;
}
</style>
