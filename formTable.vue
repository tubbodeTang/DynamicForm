<!-- 表单组表格 -->
<template>
  <div>
    <el-table
      ref="formTable"
      :key="formKey"
      :data="formModule[formKey]"
      :class="{ 'edit-table': isEdit }"
      class="form-table"
      style="width: 100%"
      @cell-mouse-enter="cellMouseEnter"
      @cell-mouse-leave="cellMouseLeave"
    >
      <!-- <template slot="empty" v-if="isEdit">
        <div class="noData">
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="mini"
            @click="addNewRow(0)"
          ></el-button>
        </div>
      </template> -->
      <el-table-column
        v-if="isEdit"
        width="16px"
        :resizable="false"
        class-name="edit-col"
      >
        <template slot-scope="scope">
          <!-- <div v-if="scope.row.hoverFlag"> -->
          <div>
            <!-- <el-button
              type="primary"
              icon="el-icon-plus"
              circle
              size="mini"
              @click="addNewRow(scope.$index)"
            ></el-button> -->
            <!-- v-if="formModule[formKey].length > 1 || scope.$index != 0" -->
            <!-- <el-button
              type="primary"
              icon="el-icon-minus"
              circle
              size="mini"
            ></el-button> -->
            <img
              class="delete-icon"
              :src="require('@@/images/table-delete.png')"
              @click="deleteRow(scope.$index)"
            />
          </div>
        </template>
      </el-table-column>
      <el-table-column
        v-for="formItem in formInfo"
        :key="formItem.key"
        :prop="formItem.key"
        :label="formItem.name"
        :align="formItem.displayAlign"
        :width="formItem.width"
      >
        <template slot-scope="scope">
          <el-form-item
            v-if="isEdit"
            class="table-form"
            :prop="formKey + '.' + scope.$index + '.' + formItem.key"
            :rules="rules[formItem.key]"
          >
            <template v-if="formItem.type == 'input'">
              <el-input v-model="scope.row[formItem.key]"
                ><i
                  slot="suffix"
                  style="font-style: normal; margin-right: 10px"
                  >{{ formItem.unit }}</i
                ></el-input
              >
              <!-- :placeholder="'请输入' + formItem.name" -->
            </template>
            <template v-if="formItem.type == 'inputArea'">
              <el-input
                v-model="scope.row[formItem.key]"
                size="mini"
                type="textarea"
                :autosize="{ minRows: 2, maxRows: 4 }"
              />
            </template>

            <template v-if="formItem.type == 'additionalSelect'">
              <addition-select
                :list-data="formOptions[formKey + '_' + formItem.options]"
                :model="scope.row[formItem.key]"
                :is-multiple="formItem.multiple"
                :key-name="formItem.optionVal"
                :label-name="formItem.optionLabel"
                :addition-name="'新增'"
                :addition-func="showAddNewDialog"
                :addition-param="
                  formKey + '.' + scope.$index + '.' + formItem.key
                "
                :show-addition="canAddNewKeys.indexOf(formItem.key) > -1"
                @selectChange="
                  ($event) => {
                    setValue(formKey, formItem.key, scope.$index, $event);
                  }
                "
              ></addition-select>
            </template>
            <template v-if="formItem.type == 'select'">
              <el-select
                v-model="scope.row[formItem.key]"
                :filterable="formItem.filterable"
                :multiple="formItem.multiple"
                :allow-create="formItem.allowCreate"
                :default-first-option="formItem.filterable"
                clearable
              >
                <i
                  slot="suffix"
                  style="font-style: normal; margin-right: 10px"
                  >{{ formItem.unit }}</i
                >
                <template
                  v-for="i in formOptions[formKey + '_' + formItem.options]"
                >
                  <el-option
                    :key="i[formItem.optionVal] || i"
                    :label="i[formItem.optionLabel] || i"
                    :value="i[formItem.optionVal] || i"
                    :disabled="i.isDisabled"
                  ></el-option>
                </template>
              </el-select>
            </template>

            <template v-if="formItem.type == 'dateRange'">
              <el-date-picker
                v-model="scope.row[formItem.key]"
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
          <span v-else>
            <template v-if="formItem.type == 'input'">{{
              scope.row[formItem.key] || '--'
            }}</template>
            <template
              v-if="
                formItem.type == 'select' || formItem.type == 'additionalSelect'
              "
            >
              {{
                scope.row[formItem.key + 'Name'] ||
                scope.row[formItem.key] ||
                '--'
              }}</template
            >
            <span class="unit">{{ formItem.unit }}</span>
          </span>
        </template>
      </el-table-column>
      <!-- 额外行-缺失说明 -->
      <div v-if="!isEdit && formModule[commentKey]" slot="append">
        <div ref="comment" class="table_appendix">
          <div class="table_appendix_unit center">
            <span>
              <span class="comment">缺失说明</span>
              <el-tooltip placement="top" class="no-print">
                <div slot="content">
                  如无法提供字段信息，请如实填写每个空格的缺失原因
                </div>
                <i class="el-icon-question"></i>
              </el-tooltip>
            </span>
          </div>
          <div class="table_appendix_unit">{{ formModule[commentKey] }}</div>
        </div>
      </div>
    </el-table>
    <el-button
      v-if="isEdit"
      type="primary"
      icon="el-icon-circle-plus-outline"
      @click="addNewRow(0)"
      >添加</el-button
    >
  </div>
</template>

<script>
import AdditionSelect from '@/components/common/additionSelect.vue';
import { getRandomUnixTimeStr } from '@/utils/index.js';
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
    // 表单下拉数据
    formOptions: {
      type: Object,
      default: function () {
        return {};
      },
    },
    // 可新增选项的项目
    canAddNewKeys: {
      type: Array,
      default: function () {
        return [];
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
    return {
      currentRow: null,
    };
  },
  computed: {},
  watch: {
    $route(to) {
      if (to.path.indexOf('Check') > -1 || to.path.indexOf('Approval') > -1) {
        setTimeout(() => {
          this.adjustAppendixWidth();
        }, 100);
      }
    },
  },
  mounted() {
    this.formModule[this.formKey] &&
      this.formModule[this.formKey].forEach((row) => {
        row.rowId = getRandomUnixTimeStr(5);
      });
    this.adjustAppendixWidth();
    window.addEventListener('resize', this.adjustAppendixWidth.bind(this));
  },
  methods: {
    addNewRow(index) {
      let newObj = {};
      this.formInfo.forEach((formItem) => {
        newObj[formItem.key] = formItem.multiple ? [] : '';
      });
      newObj.rowId = getRandomUnixTimeStr(5);
      this.formModule[this.formKey].splice(index + 1, 0, newObj);
    },
    deleteRow(index) {
      this.formModule[this.formKey].splice(index, 1);
    },

    // 监听行悬浮事件
    cellMouseEnter(curRow, column, cell, event) {
      //   console.log("移入hover事件", curRow, column, cell, event);
      this.formModule[this.formKey].forEach((row) => {
        if (row.rowId == curRow.rowId) {
          //   console.log("找到对应行");
          this.$set(row, 'hoverFlag', true);
        } else {
          this.$set(row, 'hoverFlag', false);
        }
      });
    },
    cellMouseLeave() {
      this.formModule[this.formKey].forEach((row) => {
        this.$set(row, 'hoverFlag', false);
      });
    },

    // 调整额外行宽度
    adjustAppendixWidth() {
      this.$nextTick(() => {
        if (
          this.$refs &&
          this.$refs.comment &&
          this.$refs.formTable &&
          this.$refs.formTable.$refs.headerWrapper
        ) {
          let wholeWidth =
            window.getComputedStyle(
              this.$refs.formTable.$refs.headerWrapper.querySelector('table')
            ).width ||
            this.$refs.formTable.$refs.headerWrapper.querySelector('table')
              .style.width;

          this.$refs.comment.style = 'width:' + wholeWidth;
          console.log('wholeWidth', wholeWidth);

          let tableCols = Array.from(
            this.$refs.formTable.$refs.headerWrapper.querySelectorAll('col')
          );
          let appendixCols = Array.from(this.$refs.comment.children);

          if (appendixCols.length < tableCols.length) {
            // 列数不够，合并除表头外的剩下所有单元格
            let spanWidth = 0;
            appendixCols.forEach((appendixCol, i) => {
              if (appendixCol) {
                appendixCol.style =
                  'width:' + tableCols[i].getAttribute('width') + 'px';
                if (i == appendixCols.length - 1) {
                  appendixCol.style =
                    'width:' + (parseFloat(wholeWidth) - spanWidth) + 'px';
                }
                spanWidth += parseFloat(tableCols[i].getAttribute('width'));
              }
            });
          } else {
            // 列数相同，按上面表格中的列宽设置每个单元格
            tableCols.forEach((n, i) => {
              //   console.log("列队", n, n.getAttribute("width"));
              if (appendixCols[i]) {
                appendixCols[i].style =
                  'width:' + n.getAttribute('width') + 'px';
              }
            });
          }
        }
      });
    },

    // 新增下拉项目
    showAddNewDialog(formKey) {
      this.$emit('addNewSelectItem', formKey);
    },
    setValue(formKey, formItemKey, arrayIndex, val) {
      this.$set(this.formModule[formKey][arrayIndex], formItemKey, val);
    },
  },
};
</script>

<style scoped>
>>> .el-table .el-form-item__content {
  margin-left: 0 !important;
}
</style>
