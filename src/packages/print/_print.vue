<template>
  <div class="xPagePrint" v-resize="handleResize">

    <div class="print-wrap no-print">
      <div style="height: 10px; background: #ffffff"></div>
    </div>

    <div class="print-header">
      <div class="print-wrap" id="printHeader">
        <div class="header-content" v-if="$slots.header">
          <slot name="header"></slot>
        </div>
      </div>
    </div>

    <div class="print-wrap">
      <table>
        <thead>
          <tr>
            <td>
              <div
                class="print-header-space"
                :style="'height:' + headerHeight + 'px;'"
              ></div>
            </td>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>
              <!--*** CONTENT GOES HERE ***-->
              <div class="print-content">
                <div>
                  <slot></slot>
                </div>
              </div>
              <!-- ./ end 内容区 -->
            </td>
          </tr>
        </tbody>
        <tfoot>
          <tr>
            <td>
              <div
                class="print-footer-space"
                :style="'height:' + footerHeight + 'px;'"
              ></div>
            </td>
          </tr>
        </tfoot>
      </table>
    </div>

    <div class="print-footer">
      <div class="print-wrap" id="printFooter">
        <div class="footer-content" v-if="$slots.footer">
          <slot name="footer"></slot>
        </div>
      </div>
    </div>
    <div class="print-wrap no-print">
      <div style="height: 10px; background: #ffffff"></div>
    </div>
  </div>
</template>

<style lang="less" scoped>
@printWidth: 768px;

.xPagePrint {
  white-space: normal;
  word-break: break-all;
  word-wrap: break-word;

  .print-wrap {
    margin: 0 auto;
    padding: 0px 0px;
    background: #ffffff;
  }

  .print-header,
  .print-header-space {
    .header-content {
      width: 100%;
      border-bottom: 1px solid #333333;
    }
  }

  .print-footer,
  .print-footer-space {
    .footer-content {
      width: 100%;
    }
  }

  .print-header {
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 500;
  }

  .print-footer {
    position: fixed;
    bottom: 0;
    width: 100%;
    z-index: 500;
  }

  .print-content {
    page-break-after: always;
  }

  @media print {
    input {
      display: none;
    }
    thead {
      display: table-header-group;
    }
    tfoot {
      display: table-footer-group;
    }
    body {
      margin: 0;
    }
    .no-print {
      display: none;
    }
    .no-screen {
      display: block;
    }
    .print-wrap {
      padding: 0px 0px;
      width: 100%;
    }

    .print-header {
      position: fixed;
      top: 0;
      width: 100%;
      z-index: 500;
    }
    .print-footer {
      position: fixed;
      bottom: 0;
      width: 100%;
      z-index: 500;
    }
  }

  @media screen {
    .no-print {
      display: block;
    }
    .no-screen {
      display: none;
    }
    .print-wrap {
      padding: 0px 30px;
      width: @printWidth;
    }
    .print-header-space {
      display: none;
    }
    .print-header {
      position: relative;
    }
    .print-footer-space {
      display: none;
    }
    .print-footer {
      position: relative;
    }
  }
}
</style>

<script>
import { getPosition } from "../../utils/position.js";

export default {
  data() {
    return {
      headerHeight: 0,
      footerHeight: 0,
    };
  },
  mounted() {
    this.handleResize();
  },
  computed: {
    contentStyle() {
      return `padding-top:${this.headerHeight}px; padding-bottom:${this.footerHeight}px;`;
    },
  },
  methods: {
    handleResize() {
      try {
        this.headerHeight = getPosition("#printHeader").height;
      } catch (error) {}
      try {
        this.footerHeight = getPosition("#printFooter").height;
      } catch (error) {}
    },
  },
};
</script>