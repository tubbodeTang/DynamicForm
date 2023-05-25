<template>
  <div>
    <div>
      <!-- <el-button
        v-if="isEdit && !addRowInLine"
        type="text"
        class="right"
        @click="addAttchItem"
        >增加附件</el-button
      > -->
      <el-button
        v-if="
          formModule.attachList &&
          formModule.attachList.length > 0 &&
          showBatchDownload
        "
        size="mini"
        style="float: right; margin-bottom: 5px"
        type="primary"
        class="attach-down-btn"
        @click="loadAll"
        >批量下载</el-button
      >
    </div>

    <el-table
      ref="attachRef"
      :data="formModule.attachList"
      :class="{ 'edit-table': isEdit && addRowInLine }"
      @cell-mouse-enter="cellMouseEnter"
      @cell-mouse-leave="cellMouseLeave"
    >
      <el-table-column
        v-if="isEdit && addRowInLine"
        width="25px"
        :resizable="false"
        class-name="edit-col"
      >
        <template slot-scope="scope">
          <div v-if="scope.row.hoverFlag">
            <el-button
              type="primary"
              icon="el-icon-plus"
              circle
              size="mini"
              @click="addAttchItem(scope.$index)"
            ></el-button>
            <el-button
              v-if="formModule.attachList.length > 1 || scope.$index != 0"
              type="primary"
              icon="el-icon-minus"
              circle
              size="mini"
              @click="delRowDatas(scope.$index)"
            ></el-button>
          </div>
        </template>
      </el-table-column>
      <el-table-column
        v-if="isEdit"
        header-align="center"
        label="操作"
        width="100"
        align="center"
      >
        <!-- fixed="left" -->
        <template slot-scope="scope">
          <el-button size="mini" type="text" @click="uploads(scope.$index)">{{
            !scope.row.attchName ? '上传' : '更新'
          }}</el-button>
          <el-button type="text" @click="delRowDatas(scope.$index)"
            >删除</el-button
          >
          <el-progress
            v-show="scope.row.showProgress == true"
            :key="scope.row.rowId"
            :percentage="uploadPercent"
            color="#8e71c7"
          ></el-progress>
          <el-upload
            class="upload-demo"
            :headers="headers"
            :before-upload="beforeUploads"
            :action="'/api/file/uploadFile'"
            :on-success="attchSuccessCallback"
            :show-file-list="false"
            :object-bind="scope.$index"
            :on-progress="uploadProcess"
            :on-error="attchErrorCallback"
            :accept="acceptFileType"
          >
            <el-button
              :ref="'uploadBtn' + scope.$index"
              size="mini"
              type="primary"
              >附件</el-button
            >
          </el-upload>
        </template>
      </el-table-column>
      <el-table-column
        v-if="attachTypeList.length > 0 && infoKeys.includes('style')"
        prop="style"
        label="附件类型"
        header-align="center"
        align="center"
      >
        <template slot-scope="scope">
          <el-select
            v-if="isEdit"
            v-model="scope.row.style"
            placeholder="请选择"
            style="width: 100%"
          >
            <el-option
              v-for="item in attachTypeList"
              :key="item.code"
              :label="item.info"
              :value="item.code"
            ></el-option>
          </el-select>
          <span v-else>{{ scope.row.style }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="attchName"
        label="附件名称"
        header-align="center"
        align="center"
      ></el-table-column>
      <el-table-column
        label="附件链接"
        header-align="center"
        align="center"
        width="200"
      >
        <template slot-scope="scope">
          <!-- 数字需要转换成可下载的随机id -->
          <a
            v-if="
              scope.row.attchName && host && /^[0-9]*$/.test(scope.row.attchUrl)
            "
            style="display: block"
            href="javascript:;"
            @click="downloadFile(scope.row.newId)"
            ><span>{{ host + '/api/file/downFile?id=' }}</span>
            {{ scope.row.newId }}</a
          >
          <!-- 非数字即可下载的随机id -->
          <a
            v-else-if="
              scope.row.attchName &&
              host &&
              !/^[0-9]*$/.test(scope.row.attchUrl)
            "
            style="display: block"
            href="javascript:;"
            @click="downloadFile(scope.row.attchUrl)"
          >
            <span>{{ host + '/api/file/downFile?id=' }}</span>
            {{ scope.row.attchUrl }}
          </a>
          <a
            v-else-if="scope.row.attchName && !host"
            href="javascript:;"
            @click="goOutSideLink(scope.row.newId)"
          >
            {{ scope.row.newId }}</a
          >
        </template>
      </el-table-column>
      <el-table-column
        v-if="hasDesc"
        prop="attchDesc"
        label="说明"
        header-align="center"
        align="center"
      >
        <template slot-scope="scope">
          <el-form-item
            v-if="isEdit"
            :prop="'attachList.' + scope.$index + '.' + 'attchDesc'"
            :rules="rules.attchDesc"
            class="table-form"
          >
            <el-input
              v-model.trim="scope.row.attchDesc"
              type="textarea"
              :autosize="{ minRows: 1 }"
            ></el-input>
          </el-form-item>
          <span v-else>{{ scope.row.attchDesc }}</span>
        </template>
      </el-table-column>
      <el-table-column
        prop="creator"
        header-align="center"
        align="center"
        label="上传人"
      ></el-table-column>
      <el-table-column
        prop="createDept"
        header-align="center"
        align="center"
        label="上传人部门"
      ></el-table-column>
      <el-table-column
        prop="gmtCreate"
        header-align="center"
        align="center"
        label="上传时间"
      ></el-table-column>
    </el-table>
    <el-button
      v-if="isEdit && !addRowInLine"
      type="primary"
      icon="el-icon-circle-plus-outline"
      @click="addAttchItem"
      >添加</el-button
    >
  </div>
</template>

<script>
import { downloadSingleFile, downloadBatchFile } from '@/utils/downloadFile.js';
import { formatDate, getRandomUnixTimeStr } from '@/utils/index.js';
export default {
  props: {
    formModule: {
      type: Object,
      default: function () {
        return {};
      },
    },
    rules: {
      type: Object,
      default: function () {
        return {};
      },
    },
    addRowInLine: {
      // 是否行内添加行
      type: Boolean,
      default: true,
    },
    hasDesc: {
      // 是否显示上传说明
      type: Boolean,
      default: true,
    },
    isEdit: {
      // 是否能编辑
      type: Boolean,
      default: false,
    },
    showBatchDownload: {
      // 是否能批量下载
      type: Boolean,
      default: false,
    },
    batchName: {
      // 批量下载文件名
      type: String,
      default: '批量下载附件',
    },
    acceptFileType: {
      // 接受文件类型
      type: String,
      default: function () {
        return `.pptx, .ppt,.xls,.xlsx, .xltx, .csv, .xlsb, .xlsm,.doc,
        .dot,.wps, .wpt, .docx, .dotx, .docm, .dotm, .wpss,.pdf,.jpeg, .jpg, .png, .gif, .bmp`;
      },
    },
    attachTypeList: {
      // 文件类型选项
      type: [Array, Object],
      default: function () {
        return [];
      },
    },
    businessType: {
      // 特殊业务要求
      type: String,
      default: '',
    },
    host: {
      type: String,
      default: window.location.protocol + '//' + window.location.host,
    },
  },
  data() {
    return {
      uploadPercent: 0,
      headers: null,
    };
  },
  computed: {
    infoKeys() {
      return Object.keys(
        this.formModule.attachList.length > 0 && this.formModule.attachList[0]
      );
    },
  },
  watch: {
    // 编辑时因为子组件钩子函数先于父组件挂载，所以不会触发mounted中的判断
    attachTypeList: function (newVal) {
      if (newVal && this.formModule.attachList.length < 1) {
        this.addAttchItem();
      }
    },
  },
  created() {
    this.headers = { 'Access-Token': sessionStorage.getItem('tkToken') };
  },
  mounted() {
    // 新增时传值因为父组件v-if导致子组件无法watch  暂时多加一个判断
    if (this.attachTypeList && this.attachTypeList.length > 0) {
      this.addAttchItem();
    }
  },
  methods: {
    // 上传文件大小限制100M内
    beforeUploads(file) {
      let isList100 = file.size / 1024 / 1024 < 100;
      if (!isList100) {
        this.$warnToast('文件大小不能超过100M');
        return isList100;
      }
      // let repeatStatus = true;
      // let fileStyle = this.attachList[this.attchIndex].style;
      // this.attachList.some(item => {
      //   if (item.style == fileStyle && item.attchName == file.name) {
      //     repeatStatus = false;
      //      this.$warnToast("文件名重复，请修改后再试");
      //     return;
      //   }
      // });
      // return repeatStatus;
    },
    // 增加附件
    addAttchItem(index) {
      // let attachRef = this.$refs.
      // attachRef.scrollBy(0, -100)
      if (
        (this.uploadPercent > 0 && this.uploadPercent != 100) ||
        (this.uploadPercent > 0 && !this.uploadStatus)
      ) {
        this.$warnToast('有正在上传的附件');
        return;
      }
      if (index) {
        let newObj = {
          style: '1',
          gmtCreate: '',
          attchDesc: '',
          creator: '',
          createDept: '',
          showProgress: false,
        };
        newObj.rowId = getRandomUnixTimeStr(10);
        this.formModule.attachList.splice(index + 1, 0, newObj);
        // if (this.attchIndex) {
        //   this.attchIndex++;
        // }
      } else {
        let newObj = {
          style: '1',
          gmtCreate: '',
          attchDesc: '',
          creator: '',
          createDept: '',
          showProgress: false,
        };
        newObj.rowId = getRandomUnixTimeStr(10);
        this.formModule.attachList.push(newObj);
      }
      // console.log(' this.formModule.attachList===', this.formModule.attachList);
    },
    // 下载附件
    downloadFile(id) {
      downloadSingleFile('get', id)
        .then((result) => {})
        .catch((err) => {
          this.$errToast(err || '下载失败');
        });
    },
    // 跳转外部链接
    goOutSideLink(url) {
      window.open(url);
    },
    //批量下载
    loadAll() {
      let allFiles = [];
      this.formModule.attachList.forEach((item) => {
        if (item.attchName) {
          let id = /^[0-9]*$/.test(item.attchUrl) ? item.newId : item.attchUrl;
          allFiles.push(id);
        }
      });
      downloadBatchFile('get', allFiles, this.batchName)
        .then((result) => {})
        .catch((err) => {
          this.$errToast(err || '下载失败');
        });
    },
    // 删除附件
    delRowDatas(val) {
      if (
        this.businessType !== 'productInfo' &&
        this.formModule.attachList.length <= 1
      ) {
        // this.$warnToast('请至少上传一个附件')
        this.formModule.attachList.splice(val, 1);
        let newObj = {
          style: '1',
          gmtCreate: '',
          attchDesc: '',
          creator: '',
          createDept: '',
          showProgress: false,
        };
        newObj.rowId = getRandomUnixTimeStr(10);
        this.formModule.attachList.push(newObj);
      } else {
        this.formModule.attachList.splice(val, 1);
      }
    },
    //上传进度条
    uploadProcess(event, file, fileLists, index) {
      this.uploadStatus = false;
      this.uploadPercent = Math.floor(event.percent);
      this.formModule.attachList[this.attchIndex].showProgress = true;
    },
    // 上传文件的方法
    uploads(val) {
      console.log('val----', val);
      if (
        (this.uploadPercent > 0 && this.uploadPercent != 100) ||
        (this.uploadPercent > 0 && !this.uploadStatus)
      ) {
        this.$warnToast('有正在上传的附件');
      } else {
        this.attchIndex = val;
        let btn = 'uploadBtn' + val;
        this.$refs[btn].$el.click();
      }
    },
    attchSuccessCallback(res, file, fileList, index) {
      this.formModule.attachList[this.attchIndex].showProgress = false;
      this.uploadStatus = true;
      if (res.code == 'success') {
        let time = formatDate(new Date());
        let obj = {
          style: '',
          attchName: '',
          attchUrl: '',
          newId: '',
          attchDesc: '',
          gmtCreate: '',
          creator: '',
          createDept: '',
          showProgress: false,
        };
        obj.style = this.formModule.attachList[this.attchIndex].style;
        obj.attchName = res.data.fileName;
        obj.attchUrl = res.data.id;
        obj.newId = res.data.newId;
        obj.attchDesc = this.formModule.attachList[this.attchIndex].attchDesc;
        obj.gmtCreate = time;
        obj.creator = sessionStorage.getItem('username').split(' ')[1];
        obj.createDept = sessionStorage.getItem('username').split(' ')[0];
        this.formModule.attachList.splice(this.attchIndex, 1, obj);
        this.uploadPercent = 0;
      } else {
        this.$errToast(res.message || '上传失败');
      }
    },
    attchErrorCallback(err, file, fileList) {
      this.formModule.attachList[this.attchIndex].showProgress = false;
      this.uploadPercent = 0;
      this.$errToast('上传失败');
    },
    // 监听行悬浮事件
    cellMouseEnter(curRow, column, cell, event) {
      //   console.log("移入hover事件", curRow, column, cell, event);
      this.formModule.attachList.forEach((row, index) => {
        if (row.rowId == curRow.rowId) {
          //   console.log("找到对应行");
          this.$set(row, 'hoverFlag', true);
        } else {
          this.$set(row, 'hoverFlag', false);
        }
      });
    },
    cellMouseLeave() {
      this.formModule.attachList.forEach((row, index) => {
        this.$set(row, 'hoverFlag', false);
      });
    },
  },
};
</script>

<style scoped>
.upload-demo {
  display: none;
}
>>> .el-table .el-form-item__content {
  margin-left: 0px !important;
}
.el-table .el-form-item {
  margin-bottom: 15px;
}

>>> .edit-table.el-table--border {
  border: none;
}
>>> .edit-table.el-table--border::after,
>>> .edit-table.el-table::before {
  background-color: transparent;
}
>>> .edit-table.el-table th.el-table__cell.is-leaf {
  border-top: 1px solid #ebeef5;
}

>>> .el-table .edit-col.el-table__cell {
  border-bottom: 0 !important;
  border-top: 0 !important;
  border-right: 1px solid #ebeef5;
  padding: 0;
}
>>> .el-table--border .edit-col.el-table__cell:first-child .cell {
  padding-right: 5px;
  padding-left: 0;
}
.edit-col .el-button--mini {
  scale: 0.8;
  padding: 2px;
}
.edit-col .el-button--mini + .el-button--mini {
  margin-left: 0;
}
</style>
