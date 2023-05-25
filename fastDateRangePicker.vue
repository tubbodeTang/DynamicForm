<template>
  <div class="fast-daterange-picker">
    <el-date-picker
      :picker-options="pickerOptions"
      clearable
      :disabled="disabled"
      style="width: 65%"
      @change="handleDateChange"
      v-model="time"
      :unlink-panels="true"
      type="daterange"
      range-separator="-"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
    >
    </el-date-picker>
    <span class="time-fast-picker">
      <el-radio v-model="timeFastSelect" label="week">近一周</el-radio>
      <el-radio v-model="timeFastSelect" label="month">近一个月</el-radio>
    </span>
  </div>
</template>
<script>
import { formatDate, getMonthAgo, getWeekAgo } from '@/utils/index';
export default {
  name: 'FastDateRangePicker',
  props: {
    disabled: Boolean,
  },
  data() {
    return {
      time: '',
      timeFastSelect: '',
      pickerOptions: {
        disabledDate(time) {
          return time.getTime() > Date.now();
        },
      },
    };
  },
  watch: {
    timeFastSelect(newVal) {
      if (!newVal) return;
      let times = newVal == 'week' ? getWeekAgo() : getMonthAgo();
      // times contains - {timeRange,formatAgo,formatNow}
      this.time = times.timeRange;
      this.$emit('dateChange', times);
    },
  },
  methods: {
    handleDateChange(val) {
      this.timeFastSelect = '';
      let times = {
        formatNow: '',
        formatAgo: '',
      };
      if (val) {
        times.formatAgo = formatDate(new Date(val[0]));
        times.formatNow = formatDate(new Date(val[1]));
      }
      this.$emit('dateChange', times);
    },
    clearTime() {
      this.timeFastSelect = '';
      this.time = '';
    },
  },
};
</script>
<style scoped>
.fast-daterange-picker {
  display: flex;
}
</style>
