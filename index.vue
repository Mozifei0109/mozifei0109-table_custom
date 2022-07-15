<!--
 * @作者: 陈菲
 * @文件作用: 图层字段
-->
<template>
  <div class="layer_root">
    <div class="layer_screen">
      <a-space :size="10">
        <h4>
          <SelectDept :reset="reset" @deptInfo="deptInfo" @initDeptInfo="initDeptInfo" />
        </h4>
        <div class="filter_btn">
          <a-button @click="handleResetInfo">重置</a-button>
          <a-button type="primary" @click="handleQueryInfo">查询</a-button>
        </div>
      </a-space>
    </div>
    <div>
      <a-tabs :style="{ width: '70vw' }" @change="handleCallback">
        <a-tab-pane v-for="item in typeList" :key="item.id" :tab="item.typeName" class="table_table">
          <a-spin :spinning="spinning" v-show="!layerData.length">
            <AddLayerFieldsTable
              :year="year"
              :layerData="layerData"
              :visibleAddPro="visibleAddPro"
              :loading="loading"
              @addLayerFields="addLayerFieldsApi"
            />
          </a-spin>
          <a-spin v-show="layerData.length" :spinning="spinning">
            <AddLayerFieldsTable
              :year="year"
              :layerData="layerData"
              :fieldsYear="fieldsYear"
              :visibleAddPro="visibleAddPro"
              :loading="loading"
              @addLayerFields="addLayerFieldsApi"
              @deleteLayerField="deleteLayerFields"
              :active="active"
              :activeItem="activeItem"
            />
          </a-spin>
        </a-tab-pane>
      </a-tabs>
    </div>
  </div>
</template>

<script>
// 组件依赖
import SelectDept from "@/views/grid/work/components/SelectDept.vue"
import AddLayerFieldsTable from "./AddLayerFieldsTable.vue"

// 接口依赖
import { getProjectCategoryList } from "@/api/proTypeApi"
import { getLayerFieldInfo, saveLayerFieldInfo, editLayerFieldInfo } from "@/api/layerFieldsApi"

// 状态管理依赖
import { createNamespacedHelpers } from "vuex"
const { mapState } = createNamespacedHelpers("user")
export default {
  props: {},
  data() {
    return {
      selectItem: "",
      activeClass: 0,
      reset: 0,
      deptId: "",
      level: "",
      deptLevel: "",
      typeList: [],
      fliteFrom: {
        region: "",
        activePro: "",
      },
      defaultActive: "",
      layerData: [],
      spinning: false,
      year: new Date().getFullYear(),
      fieldsYear: [],
      visibleAddPro: false,
      loading: false,
      fieldsVal: ["面积"],
      active: "",
      activeItem: {},
    }
  },
  computed: {
    ...mapState({
      userInfo: (state) => state.userInfo,
    }),
  },

  watch: {
    layerData: {
      handler(val) {
        if (val) {
          return val
        }
      },
      deep: true,
    },
  },
  mounted() {
    this.getProTypeList()
  },
  methods: {
    // 责任区
    initDeptInfo(val) {
      this.deptLevel = val.Level
      this.level = val.Level
      this.deptId = val.deptId
    },

    deptInfo(val) {
      this.deptLevel = val.Level
      this.fliteFrom.region = val.deptId
    },

    handleCallback(val) {
      this.fliteFrom.activePro = val
      this.defaultActive = val
      if (this.fliteFrom.region !== "") {
        this.getLayerFieldList(this.fliteFrom.region, val)
      } else {
        this.getLayerFieldList(this.userInfo.deptId, val)
      }
    },

    // 获取项目列表接口
    async getProTypeList() {
      const [err, data] = await getProjectCategoryList()
      if (!err) {
        this.typeList = data
        this.defaultActive = data[0].id
        this.fliteFrom.activePro = data[0].id
        this.fliteFrom.region = this.userInfo.deptId

        this.getLayerFieldList(this.userInfo.deptId, data[0].id)
      }
    },

    // 获取图层字段列表接口
    async getLayerFieldList(deptId, projectCategoryId) {
      setTimeout(() => {
        this.spinning = true
      }, 10)
      const [err, data] = await getLayerFieldInfo({ deptId: deptId, projectCategoryId: projectCategoryId })
      if (!err) {
        this.layerData = data

        const obj = []
        data.map((item) => {
          obj.push(item.year)
          this.fieldsYear = obj
        })

        this.spinning = false
      }
    },

    // 新增图层字段
    addLayerFieldsApi(val, state, year, fieldsInfo, activeItem, active) {
      if (state == "addInit") {
        this.fieldsVal.push(val.value)
        this.saveLayerFieldInfoApi(this.fieldsVal, year, this.fliteFrom)
      }
      if (state == "add") {
        this.fieldsVal = JSON.parse(fieldsInfo.value)
        this.fieldsVal.push(val.value)

        this.editLayerFieldInfoApi(fieldsInfo, this.fieldsVal)
      }
      if (state == "edit") {
        const field = JSON.parse(activeItem.value).filter(function (item) {
          return item != active
        })
        field.push(val.value)
        this.editLayerFieldInfoApi(activeItem, field)
      }
    },

    //初始新建接口
    async saveLayerFieldInfoApi(val, year, fliteFrom) {
      this.loading = true
      const [err] = await saveLayerFieldInfo({
        value: JSON.stringify(val),
        year: year,
        deptId: fliteFrom.region,
        projectCategoryId: fliteFrom.activePro,
      })
      if (!err) {
        this.$message.success("新建字段成功！")
        this.getLayerFieldList(this.fliteFrom.region, this.fliteFrom.activePro)
        this.loading = false
        this.visibleAddPro = false
      } else {
        this.$message.error("新建失败！")
        this.loading = false
      }
    },

    //再次新建接口
    async editLayerFieldInfoApi(fieldsInfo, val) {
      const [err] = await editLayerFieldInfo({
        id: fieldsInfo.id,
        value: JSON.stringify(val),
      })
      if (!err) {
        this.$message.success("新建字段成功！")
        this.getLayerFieldList(this.fliteFrom.region, this.fliteFrom.activePro)
        this.loading = false
        this.visibleAddPro = false
      } else {
        this.$message.error("新建失败！")
        this.active = ""
        this.activeItem = {}
        this.loading = false
      }
    },

    //删除字段
    deleteLayerFields(val, innerItem) {
      const field = JSON.parse(val.value).filter(function (item) {
        return item != innerItem
      })
      this.editLayerFieldInfoApi(val, field)
    },
    // 查询
    handleQueryInfo() {
      this.getLayerFieldList(this.fliteFrom.region, this.fliteFrom.activePro)
    },
    // 重置
    handleResetInfo() {
      this.fliteFrom = {
        region: "",
        activePro: this.defaultActive,
      }
      this.getLayerFieldList(this.userInfo.deptId, this.defaultActive)

      this.reset = this.reset + 1
    },
  },
  components: { SelectDept, AddLayerFieldsTable },
}
</script>

<style scoped lang="less">
@import url(../utils/style.less);

.layer_root {
  .layer_screen {
    margin-top: 21px;
    margin-bottom: 31px;
    .filter_btn {
      position: relative !important;
      top: -5px !important;
      .ant-btn:first-child {
        margin-right: 12px;
      }
      .ant-btn {
        border-radius: 3px;
      }
    }
  }
  .table_table {
    overflow: auto;
    width: 70vw;
    height: 600px;
  }
}
</style>
