<template>
  <div>
    <api-list-container
      :is-api-list-enable="isApiListEnable"
      @isApiListEnableChange="isApiListEnableChange">

      <template>
        <slot name="version"></slot>
      </template>

      <api-table-list
        :table-data="tableData"
        :version-filters="versionFilters"
        :version-enable="versionEnable"
        :project-id="projectId"
        :condition="condition"
        :select-node-ids="selectNodeIds"
        :result="result"
        :total="total"
        :current-protocol="currentProtocol"
        :screen-height="screenHeight"
        @setSelectRow="setSelectRow"
        @selectCountChange="selectCountChange"
        @refreshTable="initTable"
        ref="apitable">

        <template v-slot:header>
          <ms-environment-select :project-id="projectId" v-if="isTestPlan || isScript" :is-read-only="isReadOnly"
                                 @setEnvironment="setEnvironment" ref="msEnvironmentSelect"/>
        </template>

      </api-table-list>

    </api-list-container>

  </div>

</template>

<script>

import ApiListContainer from "../../../definition/components/list/ApiListContainer";
import MsEnvironmentSelect from "../../../definition/components/case/MsEnvironmentSelect";
import {buildBatchParam} from "@/common/js/tableUtils";
import {
  TEST_PLAN_RELEVANCE_API_DEFINITION_CONFIGS,
} from "@/business/components/common/components/search/search-components";
import ApiTableList from "@/business/components/api/definition/components/complete/ApiTableList";

export default {
  name: "RelevanceApiList",
  components: {
    ApiTableList,
    MsEnvironmentSelect,
    ApiListContainer,
  },
  data() {
    return {
      condition: {
        components: TEST_PLAN_RELEVANCE_API_DEFINITION_CONFIGS
      },
      result: {},
      screenHeight: 'calc(100vh - 400px)',//屏幕高度,
      tableData: [],
      environmentId: "",
      total: 0,
      selectRows: new Set(),
    };
  },
  props: {
    currentProtocol: String,
    selectNodeIds: Array,
    versionFilters: Array,
    currentVersion: String,
    visible: {
      type: Boolean,
      default: false,
    },
    isScript: {
      type: Boolean,
      default() {
        return false;
      }
    },
    isApiListEnable: {
      type: Boolean,
      default: false,
    },
    isReadOnly: {
      type: Boolean,
      default: false
    },
    isCaseRelevance: {
      type: Boolean,
      default: false,
    },
    projectId: String,
    planId: String,
    isTestPlan: Boolean,
    versionEnable: {
      type: Boolean,
      default: false
    }
  },
  created() {
    this.condition.versionId = this.currentVersion;
    this.initTable();
  },
  watch: {
    selectNodeIds() {
      this.initTable();
    },
    currentProtocol() {
      this.initTable();
    },
    projectId() {
      this.condition.versionId = null;
      this.initTable();
    },
    currentVersion() {
      this.condition.versionId = this.currentVersion;
      this.initTable();
    }
  },
  methods: {
    setSelectRow(setSelectRow) {
      this.selectRows = setSelectRow;
    },
    selectCountChange(data) {
      this.$emit('selectCountChange', data);
    },
    isApiListEnableChange(data) {
      this.$emit('isApiListEnableChange', data);
    },
    initTable(projectId) {
      this.condition.moduleIds = this.selectNodeIds;
      if (this.trashEnable) {
        this.condition.filters = {status: ["Trash"]};
        this.condition.moduleIds = [];
      }
      if (projectId != null && typeof projectId === 'string') {
        this.condition.projectId = projectId;
      } else if (this.projectId != null) {
        this.condition.projectId = this.projectId;
      }

      if (this.currentProtocol != null) {
        this.condition.protocol = this.currentProtocol;
      } else {
        this.condition.protocol = "HTTP";
      }
      if (this.condition.filters) {
        this.condition.filters.status = ["Prepare", "Underway", "Completed"];
      } else {
        this.condition.filters = {status: ["Prepare", "Underway", "Completed"]};
      }
      let url = '/api/definition/list/';
      if (this.isTestPlan) {
        url = '/api/definition/list/relevance/';
        this.condition.planId = this.planId;
      }

        this.$nextTick(() => {
          this.result = this.$post(url + this.$refs.apitable.currentPage + "/" + this.$refs.apitable.pageSize, this.condition, response => {
            this.total = response.data.itemCount;
            this.tableData = response.data.listObject;
            this.tableData.forEach(item => {
              if (item.tags && item.tags.length > 0) {
                item.tags = JSON.parse(item.tags);
              }
            });
          });
        });
      },
      setEnvironment(data) {
        this.environmentId = data.id;
      },
      getConditions() {
        let sampleSelectRows = this.selectRows;
        let param = buildBatchParam(this, undefined, this.projectId);
        param.ids = Array.from(sampleSelectRows).map(row => row.id);
        let tableDataIds = Array.from(this.tableData).map(row => row.id);
        param.ids.sort((a, b) => {
          return tableDataIds.indexOf(a) - tableDataIds.indexOf(b);
        });
        return param;
      },
      clear() {
        if (this.$refs.apitable) {
          this.$refs.apitable.clear();
        }
      },
      clearEnvAndSelect() {
        this.environmentId = "";
        if (this.$refs.msEnvironmentSelect) {
          this.$refs.msEnvironmentSelect.environmentId = "";
        }
        this.clear();
      }
    },
  }
</script>

<style scoped>
.card-content >>> .el-button-group {
  float: left;
}
</style>
