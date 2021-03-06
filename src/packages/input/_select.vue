<template>
  <span v-loading="configLoading">
    <template v-if="!configLoading && configInfo">
      <el-popover
        v-model="visible"
        :disabled="!!disabled"
        :visible-arrow="false"
        :placement="getValue('placement', configInfo) || $xUIPlacement"
        :width="getValue('width', configInfo)"
        trigger="click"
      >
        <div class="xInputSelect">
          <div class="xClose" @click="hideDialog">
            <i class="el-icon-close"></i>
          </div>
          <div class="xTitle">
            {{ getTitleLabel() || "请选择" }}
          </div>
          <x-table
            ref="table"
            v-if="visible"
            paginationLayout="total, prev, pager, next"
            @selectList="handleSelectList"
            :selection="multipleable"
            :tag="getValue('tag', configInfo)"
            :listApi="getValue('listApi', configInfo)"
            :btnList="getValue('btnList', configInfo)"
            :fieldList="getValue('fieldList', configInfo)"
            :paramList="getValue('paramList', configInfo)"
            :retryLabel="getValue('retryLabel', configInfo)"
            :emptyLabel="getValue('emptyLabel', configInfo)"
            :searchLabel="getValue('searchLabel', configInfo)"
            :actionLabel="getValue('actionLabel', configInfo)"
            :actionFixed="getValue('actionFixed', configInfo)"
            :defaultSort="getValue('defaultSort', configInfo)"
            :defaultParams="getValue('defaultParams', configInfo)"
            :filterMethod="filterMethod"
            :filterLabelMethod="filterLabelMethod"
            :actionList="getActionList()"
            :actionWidth="90"
          />
          <div class="xFooter" v-if="multipleable">
            <el-button @click="handleCancelClick" :size="size">
              {{ getValue("cancelLabel", configInfo) || "取消" }}
            </el-button>
            <el-button
              type="primary"
              @click="handleOkClick"
              :disabled="!(selectList && selectList.length > 0)"
              :size="size"
            >
              {{ getValue("confirmLabel", configInfo) || "确定" }}
            </el-button>
          </div>
        </div>
        <template slot="reference">
          <el-button
            v-if="getValue('showType', configInfo) == 'button'"
            :size="size"
            :style="
              styleValue || getValue('styleValue', configInfo) || 'width: 100%'
            "
            :loading="detailLoading"
            :disabled="!!disabled"
            @click="handleOpenClick"
          >
            {{ getValueLabel() || getPlaceholderLabel() }}
          </el-button>
          <el-input
            v-else
            :size="size"
            :value="getValueLabel()"
            :placeholder="getPlaceholderLabel()"
            :style="
              styleValue || getValue('styleValue', configInfo) || 'width: 100%'
            "
            :clearable="!!clearable"
            :disabled="!!disabled"
            @click.native="handleOpenClick"
            @clear="handleClearClick"
          >
            <i
              slot="prefix"
              v-if="detailLoading"
              class="el-input__icon el-icon-loading"
            ></i>
          </el-input>
        </template>
      </el-popover>
    </template>
    <template v-else-if="!configLoading && configErrorMsg">
      <el-tag
        type="danger"
        :style="
          styleValue || getValue('styleValue', configInfo) || 'width: 100%'
        "
      >
        {{ configErrorMsg || "获取选择器配置" }}
      </el-tag>
    </template>
  </span>
</template>

<style lang="less">
.xInputSelect {
  .xClose {
    float: right;
    padding: 2px 0;
    font-size: 14px;
    cursor: pointer;
    color: #919191;
  }

  .xClose:hover {
    color: #676767;
  }

  .xTitle {
    padding: 2px 0;
    font-size: 14px;
    font-weight: 600;
    color: #333;
  }

  .xFooter {
    margin-top: 10px;
    text-align: right;
  }
}
</style>

<script>
import { deepAssign } from "../../utils/assign";

export default {
  model: {
    prop: "value",
    event: "change",
  },
  props: {
    value: {
      type: String | Number,
    },
    size: {
      type: String,
      default: "",
    },
    type: {
      type: String,
      default: "",
    },
    styleValue: {
      type: String,
      default: "",
    },
    filterMethod: {
      type: Function,
    },
    filterLabelMethod: {
      type: Function,
    },
    options: {
      type: Object,
      default: () => {},
    },
    multipleable: {
      type: Boolean | String | Number,
      default: false,
    },
    clearable: {
      type: Boolean | String | Number,
      default: false,
    },
    disabled: {
      type: Boolean | String | Number,
      default: false,
    },
  },
  data() {
    return {
      select: null,
      visible: false,
      selectList: null,
      configInfo: null,
      configLoading: false,
      configErrorMsg: null,
      detailErrorMsg: null,
      detailLoading: false,
    };
  },
  watch: {
    type() {
      this.visible = false;
      this.selectList = [];
      this.refreshConfig();
    },
    options() {
      this.visible = false;
      this.selectList = [];
      this.refreshConfig();
    },
    value() {
      this.fetchDetail();
    },
    visible() {
      if (!this.visible) {
        this.$emit("close");
      }
    },
    select() {
      if (this.select) {
        let selectValue = this.selectValue;
        if (selectValue != this.value) {
          this.$emit("change", selectValue);
        }
        this.$emit("select", this.select);
        this.dispatch("ElFormItem", "el.form.change");
      } else if (this.value) {
        this.$emit("change", "");
        this.$emit("select", null);
        this.dispatch("ElFormItem", "el.form.change");
      }
    },
  },
  computed: {
    selectValue() {
      if (!this.select) {
        return;
      }
      return this.getValue("value", this.configInfo, this.select);
    },
  },
  methods: {
    focus() {
      this.handleOpenClick();
    },
    hideDialog() {
      this.visible = false;
    },
    showDialog() {
      this.visible = true;
    },
    handleOpenClick() {
      this.selectList = [];
      this.showDialog();
      this.$nextTick(() => {
        if (this.$refs.table) {
          this.$refs.table.clearSelectList();
        }
      });
    },
    dispatch(componentName, eventName, params) {
      setTimeout(() => {
        try {
          var parent = this.$parent || this.$root;
          var name = parent.$options.componentName;
          while (parent && (!name || name !== componentName)) {
            parent = parent.$parent;
            if (parent) {
              name = parent.$options.componentName;
            }
          }
          if (parent) {
            parent.$emit.apply(parent, [eventName].concat(params));
          }
        } catch (error) {}
      }, 200);
    },
    getValue(prop, item, param, def) {
      if (!item || !prop) {
        return def;
      }
      let propValue = item[prop];
      if (propValue && typeof propValue === "function") {
        return propValue(param, this.value);
      }
      if (typeof propValue === "undefined") {
        return def;
      }
      return propValue;
    },
    getActionList() {
      return [
        {
          label: (row, tag, table) => {
            return this.getValue("value", this.configInfo, row) ==
              this.selectValue
              ? this.getValue("currentLabel", this.configInfo) || "当前"
              : this.getValue("selectLabel", this.configInfo) || "选择";
          },
          linkType: (row, tag, table) => {
            return this.getValue("value", this.configInfo, row) ==
              this.selectValue
              ? "info"
              : "primary";
          },
          click: (row, refresh, table) => {
            this.handleSelectClick(row);
          },
        },
      ];
    },
    getValueLabel() {
      if (!this.select) {
        return;
      }
      return (
        this.getValue("renderLabel", this.configInfo) ||
        this.getValue("render", this.configInfo, this.select)
      );
    },
    getTitleLabel() {
      return (
        this.getValue("titleLabel", this.configInfo) ||
        this.getValue("title", this.configInfo)
      );
    },
    getPlaceholderLabel() {
      if (this.detailErrorMsg) {
        return this.detailErrorMsg;
      }
      return this.getTitleLabel();
    },
    handleSelectList(list) {
      this.selectList = list;
    },
    handleClearClick() {
      this.handleSelectClick(null);
    },
    handleSelectClick(row) {
      this.select = row;
      this.hideDialog();
    },
    handleCancelClick() {
      this.hideDialog();
    },
    handleOkClick() {
      (this.selectList || []).forEach((item) => {
        this.$emit("select", item);
      });
      if (this.$refs.table) {
        this.$refs.table.clearSelectList();
      }
      this.hideDialog();
    },
    fetchDetail() {
      if (this.select) {
        return;
      }
      if (this.detailLoading) {
        return;
      }
      if (!this.configInfo) {
        return;
      }
      if (!this.value) {
        this.select = null;
        this.detailErrorMsg = null;
        return;
      }
      this.$nextTick(() => {
        this.select = null;
        this.detailLoading = true;
        this.detailErrorMsg = null;
        let detailApi = this.getValue("detailApi", this.configInfo);
        let params =
          this.getValue("params", this.configInfo, this.value) || this.value;
        return this.$xUIDataDetailHandler(detailApi, params)
          .then((resp) => {
            this.select = resp;
          })
          .catch((err) => {
            console.error(err);
            this.detailErrorMsg = err || "请求数据失败";
          })
          .finally(() => {
            this.detailLoading = false;
          });
      });
    },
    refreshConfig() {
      if (this.configLoading) {
        return;
      }
      this.$nextTick(() => {
        this.configInfo = null;
        this.configLoading = true;
        this.configErrorMsg = null;
        return this.$xUIDataConfigHandler(this.type)
          .then((resp) => {
            if (resp) {
              this.configInfo = deepAssign({}, resp, this.options);
              this.fetchDetail();
            } else {
              throw "找不到选择器配置信息";
            }
          })
          .catch((err) => {
            console.error(err);
            this.configErrorMsg = err || "请求数据失败";
          })
          .finally(() => {
            this.configLoading = false;
          });
      });
    },
  },
  created() {
    this.refreshConfig();
  },
};
</script>