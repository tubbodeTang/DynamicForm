<template>
  <!-- 多行重复递归 -->
  <el-table-column
    v-if="coloumnHeader.group && coloumnHeader.group.length"
    :label="coloumnHeader.name"
    :fixed="coloumnHeader.freeze"
    :width="coloumnHeader.width"
    :align="coloumnHeader.align"
    :label-class-name="coloumnHeader.labelClass"
    :class-name="coloumnHeader.class"
  >
    <template v-for="item in coloumnHeader.group">
      <tableColumn :key="item.name" :coloumn-header="item"></tableColumn>
    </template>
  </el-table-column>
  <!-- 单行 -->
  <el-table-column
    v-else
    :key="coloumnHeader.name"
    :label="coloumnHeader.name"
    :prop="coloumnHeader.key"
    :fixed="coloumnHeader.freeze"
    :width="coloumnHeader.width"
    :align="coloumnHeader.align"
    :label-class-name="coloumnHeader.labelClass"
    :class-name="coloumnHeader.class"
    show-overflow-tooltip
  >
    <template slot="header">
      <span class="header-title">{{ coloumnHeader.name }}</span>
      <el-button
        v-if="coloumnHeader.collapse !== undefined"
        class="table-header-collapse-btn"
        type="text"
        icon="el-icon-d-arrow-left"
        @click.stop="changeColumn(coloumnHeader, 'collapse')"
      ></el-button>
      <el-button
        v-if="coloumnHeader.expand === false"
        style="color: #999"
        class="table-header-collapse-btn"
        type="text"
        icon="el-icon-d-arrow-right"
        @click.stop="changeColumn(coloumnHeader, 'expand')"
      ></el-button>
    </template>
    <template slot-scope="scope">
      <!-- 一组数据处理成数组后分条展示(如幸福有约件数政策) -->
      <template
        v-if="
          Object.prototype.toString.call(scope.row[coloumnHeader.key]) ==
          '[object Array]'
        "
      >
        <span
          v-for="cellItem in scope.row[coloumnHeader.key]"
          :key="cellItem"
          :class="{ 'item-label': cellItem != '--' }"
          >{{ cellItem }}</span
        >
      </template>
      <span v-else :data-value="scope.row[coloumnHeader.key]">{{
        scope.row[coloumnHeader.key]
      }}</span>
    </template>
  </el-table-column>
</template>

<script>
export default {
  name: 'TableColumn',
  props: {
    coloumnHeader: {
      type: Object,
      required: true,
    },
  },
  computed: {},
  methods: {
    changeColumn(item, type) {
      eventBus.$emit('columnChange', { key: item.key, type: type });
      // console.log(item);
    },
  },
};
</script>

<style>
.normal-td {
  font-weight: normal;
}
</style>
