<template>
  <el-select
    ref="addictionSelect"
    filterable
    :disabled="disabled"
    :value="model"
    :value-key="keyName"
    :multiple="isMultiple"
    @change="selectChange($event)"
    @visible-change="(v) => visibleChange(v, 'addictionSelect')"
  >
    <el-option
      v-for="item in listData"
      :key="item[keyName]"
      :label="labelFomatter ? labelFomatter(item) : item[labelName]"
      :value="item[keyName]"
      :disabled="item.isDisabled"
    >
    </el-option>
  </el-select>
</template>
<script>
export default {
  name: 'AdditionSelect',
  props: {
    disabled: Boolean,
    isMultiple: Boolean,
    listData: Array,
    model: [String, Number, Array],
    labelName: String,
    keyName: String,
    labelFomatter: Function,
    showAddition: {
      // 是否显示新增项
      type: Boolean,
      default: true,
    },
    additionName: String,
    additionFunc: Function, // 添加项事件
    additionParam: [String, Object, Array], // 添加事件的传参
  },
  data() {
    return {};
  },

  methods: {
    selectChange(e) {
      this.$emit('selectChange', e);
    },
    visibleChange(visible, refName) {
      if (visible) {
        const ref = this.$refs[refName];
        console.log('ref=====', ref);
        let popper = ref.$refs.popper;
        if (popper.$el) popper = popper.$el;
        if (
          // 需要增加且之前无此按钮 - 添加按钮
          this.showAddition &&
          !Array.from(popper.children).some(
            (v) => v.className === 'el-template-menu__list'
          )
        ) {
          const el = document.createElement('ul');
          el.className = 'el-template-menu__list';
          el.style =
            'border-top:2px solid rgb(219 225 241); padding:0; color:rgb(64 158 255);font-size: 13px';
          el.innerHTML =
            `<li class="el-cascader-node text-center" style="height:37px;line-height: 37px;
            padding-left: 5px;
            font-size: 14px;
            color: #1e90ff;
            font-weight:bold;
            cursor:pointer;">
                <span class="el-cascader-node__label">
                <i class="font-blue el-icon-plus"></i>&nbsp;` +
            this.additionName +
            `</span>
            </li>`;
          popper.appendChild(el);
          el.onclick = () => {
            // 调用添加项方法并传参
            this.additionFunc(this.additionParam);
            if (ref.toggleDropDownVisible) {
              ref.toggleDropDownVisible(false);
            } else {
              ref.visible = false;
            }
          };
        } else if (
          // 无需增加且之前有此按钮 - 移除按钮
          !this.showAddition &&
          Array.from(popper.children).some(
            (v) => v.className === 'el-template-menu__list'
          )
        ) {
          popper.removeChild(
            Array.from(popper.children).find(
              (v) => v.className === 'el-template-menu__list'
            )
          );
        }
      }
    },
  },
};
</script>
