<!--
 * @作者: 陈菲
 * @文件作用: 图层字段表格
-->
<template>
  <div>
    <table v-show="!layerData.length">
      <tr>
        <th>{{ year }}</th>
        <td>
          <span>面积</span>
        </td>
        <td>
          <a-button icon="plus" type="link" @click="handleAddLayerField">新建</a-button>
        </td>
      </tr>
    </table>
    <table v-show="layerData.length && !fieldsYear.includes(year.toString())">
      <tr>
        <th>{{ year }}</th>
        <td>
          <span>面积</span>
        </td>
        <td>
          <a-button icon="plus" type="link" @click="handleAddLayerField">新建</a-button>
        </td>
      </tr>
    </table>
    <div v-show="layerData.length">
      <div
        v-for="item of layerData"
        :key="item.id"
        :style="{ width: (JSON.parse(item.value).length + 3) * 150 + 'px' }"
      >
        <table>
          <tr>
            <th>{{ item.year }}</th>
            <td
              v-for="(innerItem, index) of JSON.parse(item.value)"
              :key="index"
              @click="handleEditField(item, innerItem)"
              :class="active == innerItem && item.year == year && innerItem !== '面积' ? 'active' : ''"
            >
              <div v-if="innerItem !== '面积' && item.year == year.toString()">
                <a-tag closable @close="(e) => preventDefault(item, innerItem, e)">
                  {{ innerItem }}
                </a-tag>
              </div>
              <div v-if="innerItem !== '面积' && item.year !== year.toString()">
                <a-tag> {{ innerItem }}</a-tag>
              </div>

              <span v-if="innerItem === '面积'">
                {{ innerItem }}
              </span>
            </td>
            <td v-if="item.year == year">
              <a-button icon="plus" type="link" @click="handleAddLayerFields(item)">新建</a-button>
            </td>
            <td v-if="item.year == year">
              <a-button icon="plus" type="link" @click="handleEditItem">编辑</a-button>
            </td>
          </tr>
        </table>
      </div>
    </div>
    <!-- 新建 -->
    <AddModule
      :visible="visibleAddPro"
      v-if="visibleAddPro"
      @closeAddPro="handleClosePro"
      :state="state"
      :loading="loading"
      @handleFieldApi="handleFieldApi"
      :editVal="active"
    />
  </div>
</template>

<script>
import AddModule from "./AddModule.vue"
export default {
  props: ["year", "layerData", "fieldsYear", "loading", "visibleAddPro", "active", "activeItem"],
  data() {
    return {
      state: "",
      fieldsVal: {},
    }
  },
  computed: {},

  watch: {
    layerData: {
      handler(val) {
        if (val) {
          return val
        }
      },
      deep: true,
    },
    fieldsYear: {
      handler(val) {
        if (val) {
          return val
        }
      },
      deep: true,
    },
  },
  methods: {
    // 新增图层字段（初始）
    handleAddLayerField() {
      this.state = "addInit"
      this.active = ""
      this.activeItem = {}
      this.visibleAddPro = true
    },
    handleAddLayerFields(val) {
      this.state = "add"
      this.active = ""
      this.activeItem = {}
      this.visibleAddPro = true
      this.fieldsVal = val
    },

    preventDefault(item, innerItem, e) {
      e.preventDefault()

      this.$confirm({
        title: "删除确认",
        content: "您确定要删除该字段名称吗!",
        centered: true,

        onOk: () => {
          this.$emit("deleteLayerField", item, innerItem)
        },
      })
    },

    //编辑
    handleEditField(val, item) {
      this.active = item
      this.activeItem = val
    },

    handleEditItem() {
      if (!Object.keys(this.activeItem).length) {
        this.$message.warning("请选择要编辑的字段名称！")
      } else {
        this.visibleAddPro = true
        this.state = "edit"
      }
    },
    handleClosePro() {
      this.visibleAddPro = false
      this.active = ""
      this.activeItem = {}
    },
    handleFieldApi(val, state) {
      if (val) {
        this.$emit("addLayerFields", val, state, this.year, this.fieldsVal, this.activeItem, this.active)
      }
    },
  },
  components: { AddModule },
}
</script>

<style scoped lang="less">
table {
  border-collapse: separate;
  border-spacing: 0;
  border-radius: 5px;
  border: solid 1px #dfdfdf;
  text-align: center;
  margin-bottom: 12px;
  cursor: pointer;

  th {
    width: 150px;
    height: 34px;
  }
  td {
    border-left: solid 1px #e6e6e6;
    width: 150px;
    height: 34px;
  }
  td:hover {
    background: #e6f8e3;
  }
  .operation {
    width: 150px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  .active {
    background: #e6f8e3;
  }
  /deep/.ant-tag {
    font-size: 16px;
    border: none;
    background: #ffffff;
  }
}
</style>
