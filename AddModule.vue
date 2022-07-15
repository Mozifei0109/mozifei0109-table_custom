<template>
  <div class="pro_modal">
    <a-modal
      :title="state == 'addInit' || state == 'add' ? '新增' : '编辑'"
      :visible="visible"
      :width="738"
      @cancel="handleCancel"
      :maskClosable="false"
      :footer="null"
    >
      <a-form :form="form" :label-col="{ span: 6 }" :wrapper-col="{ span: 15 }">
        <a-form-item label="图层字段名称">
          <a-input
            class="pro_btn"
            placeholder="图层字段名称"
            v-decorator="['value', { rules: [{ required: true, whitespace: true, message: '图层字段名称' }] }]"
          />
        </a-form-item>
        <a-form-item class="clear_btn">
          <a-button class="clear_btnO" @click="handleCancel">取消</a-button>
          <a-button type="primary" @click="handleSure" :loading="loading">确定</a-button>
        </a-form-item>
      </a-form>
    </a-modal>
  </div>
</template>

<script>
export default {
  props: ["visible", "state", "loading", "editVal"],

  data() {
    return {
      form: this.$form.createForm(this),
      proAddVal: {},
    }
  },
  computed: {},
  watch: {
    editVal: {
      handler(val) {
        // 同步处理数据会导致表单找不到对应的fieldValue，so用$nextTick
        this.$nextTick(() => {
          this.form.setFieldsValue({
            value: this.editVal ?? "",
          })
        })
      },
      deep: true,
      immediate: true,
    },
  },

  mounted() {
    if (Object.keys(this.editVal).length) {
      this.form.setFieldsValue({ typeName: this.editVal.value })
    }
  },
  methods: {
    handleCancel() {
      this.$emit("closeAddPro", false)
      this.form.resetFields()
    },

    handleSure() {
      this.form.validateFields((err, value) => {
        if (!err) {
          this.$emit("handleFieldApi", value, this.state)
        }
      })
    },
  },
  components: {},
}
</script>

<style scoped lang="less">
@import url(../utils/style.less);
/deep/.ant-modal {
  top: 282px !important;
}
/deep/.ant-modal-content {
  border-radius: 13px !important;
}
/deep/.ant-modal-header {
  border-radius: 13px !important;
}
.clear_btn {
  position: relative;
  right: -22.6041667vw;
  .clear_btnO {
    margin-right: 12px;
  }
}
</style>
