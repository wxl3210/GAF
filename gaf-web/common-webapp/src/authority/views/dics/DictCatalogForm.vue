<template>
  <div class="page-container">
    <template>
      <a-breadcrumb separator=">">
        <span class="vertical-line">| </span>
        <a-breadcrumb-item class="text-bolder">{{ title }}</a-breadcrumb-item>
      </a-breadcrumb>
    </template>
    <div>
      <a-form
        :form="addOrEditForm"
        :label-col="{ span: 5 }"
        :wrapper-col="{ span: 10}"
        layout="horizontal"
      >
        <a-form-item v-show="false" label="目录分类">
          <a-input
            v-decorator="[
              'type',
              {
                rules: [
                  {
                    required: true
                  }
                ],
                initialValue: '7'
              }
            ]"
            allow-clear
            placeholder="请选择类别"
          >
            <a-select-option value="7">字典分组</a-select-option>
          </a-input>
        </a-form-item>
        <a-form-item label="所属分组">
          <a-tree-select
            :disabled="true"
            v-decorator="[
              'parentId',
              {
                rules: [
                  {
                    required: true,
                    message: '所属分组不能为空'
                  }
                ],
                initialValue: '0'
              }
            ]"
            :treeData="treeDataWithRoot"
            placeholder="请选择所属分组"
            tree-node-filter-prop="title"
            show-search
            tree-default-expand-all
            allow-clear
          >
          </a-tree-select>
        </a-form-item>
        <a-form-item label="分组名称">
          <a-input
            v-decorator="[
              'name',
              {
                rules: [
                  {
                    required: true,
                    message: '请输入目录名称'
                  }
                ]
              }
            ]"
            placeholder="请输入目录名称"
            allow-clear
          />
        </a-form-item>
        <a-form-item label="排序序号">
          <a-input-number
            :disabled="operation === 'detail'"
            :precision="0"
            :min="1"
            v-decorator="['sortSn']"
          />
        </a-form-item>
        <a-form-item label="备注">
          <a-textarea
            :disabled="operation === 'detail'"
            v-decorator="['description']"
            placeholder="请输入备注"
            auto-size
          />
        </a-form-item>
      </a-form>
    </div>
    <div class="form-foot">
      <!-- <button
        @click="deleteData"
        v-if="operation === 'edit'"
        class="cancel-modal"
        >删除</button
      >
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp; -->
      <button @click="submitForm" class="submit-gray">
        {{ submitButtonText }}
      </button>
      &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
      <button v-if="operation === 'add'" @click="handleBack" class="submit-gray">
        取消
      </button>
    </div>
  </div>
</template>

<script>
import treeUtil from '../../utils/tree/TreeUtil.js'

export default {
  props: {
    // 当parentId为'0'或为空时表示一级分类目录
    editData: {
      type: Object,
      default: null
    },
    // add表示新增 edit表示表示编辑 detail表示详情
    operation: {
      type: String,
      default: 'add'
    }
  },
  data() {
    return {
      dataId: null,
      treeData: []
    }
  },
  computed: {
    submitButtonText() {
      if (this.operation === 'edit') {
        return '保存'
      } else if (this.operation === 'add') {
        return '添加'
      } else {
        return ''
      }
    },
    treeDataWithRoot() {
        const rootNode = [{key: '0',title: '字典根目录',value:'0'}]
        return rootNode.concat(this.treeData)
    },
    title() {
      if (this.operation === 'edit') {
        return '编辑分组'
      } else if (this.operation === 'add') {
        return '新增分组'
      } else {
        return ''
      }
    }
  },
  watch: {
    editData: function(newEditData) {
      // newEditData.parentId = newEditData.catalogId
      if (this.operation === 'edit'){
        this.dataId = newEditData.catalogId
        delete newEditData.catalogId
      }

      delete newEditData.code
      delete newEditData.status
      delete newEditData.createdTime
      delete newEditData.createdBy
      delete newEditData.updatedTime
      delete newEditData.updatedBy
      delete newEditData.tenantId
      delete newEditData.sysComponentId
      this.addOrEditForm.setFieldsValue({ ...newEditData })
      if(this.dataId) {
        this.disableNodeWithChlidren(this.treeData, this.dataId)
      } else {
        this.clearDisable(this.treeData)
      }
    }
  },
  created() {
    this.getTreeDataAndSet()
  },
  beforeMount() {
    this.addOrEditForm = this.$form.createForm(this, { name: 'addOrEditForm' })
  },
  mounted() {
    // this.editData.parentId = this.editData.catalogId
    delete this.editData.code
    delete this.editData.status
    delete this.editData.createdTime
    delete this.editData.createdBy
    delete this.editData.updatedTime
    delete this.editData.updatedBy
    delete this.editData.tenantId
    delete this.editData.sysComponentId
    if (this.operation === 'edit'){
      this.dataId = this.editData.catalogId
      delete this.editData.catalogId
      this.addOrEditForm.setFieldsValue({ ...this.editData })
    } else if(this.operation === 'add') {
      this.dataId = null
      if (!this.editData.catalogId){
        this.editData.catalogId = '1'
      }
      this.addOrEditForm.setFieldsValue({parentId: this.editData.catalogId})
    }

    if(this.dataId) {
      this.disableNodeWithChlidren(this.treeData, this.dataId)
    }
  },
  methods: {
    submitForm() {
      this.addOrEditForm.validateFields(async (err) => {
        if (err) {
          event.preventDefault()
          return false
        }
        let url = `/sys-mgt/sys-catalogs`
        const data = this.addOrEditForm.getFieldsValue()
        if (this.dataId) {
          url = url + '/' + this.dataId
          const { name, sortSn, description, parentId } = data
          const rst = await this.$axios.put(url, {
            name,
            sortSn,
            description,
            parentId
          })
          if (rst.data.isSuccessed) {
            await this.getTreeDataAndSet()
            this.$message.success('更新成功')
            this.$emit('update-success', {...rst.data.data})
          } else {
            this.$message.error(`更新失败,原因:${rst.data.message}`)
          }
        } else {
          const rst = await this.$axios.post(url, data)
          if (rst.data.isSuccessed) {
            await this.getTreeDataAndSet()
            this.$message.success('添加成功')
            this.$emit('add-success', {...rst.data.data})
          } else {
            this.$message.error(`添加失败,原因:${rst.data.message}`)
          }
        }
      })
    },
    async deleteData() {
      if (this.dataId) {
          const url = `/sys-mgt/sys-catalogs/` + this.dataId
          const rst = await this.$axios.delete(url)
          if (rst.data.isSuccessed) {
            this.$message.success('删除成功')
            this.$emit('delete-success', {...rst.data.data})
            await this.getTreeDataAndSet()
            this.clear()
          } else {
            this.$message.error('删除失败，原因: ' + rst.data.message)
          }
      }
    },
    resetFormFields() {
      this.addOrEditForm.resetFields()
    },
    // setFormFields(fieldsValue) {
    //   if (fieldsValue) {
    //     this.addOrEditForm.setFieldsValue(fieldsValue)
    //   }
    // },
    clear() {
      this.dataId = null
      this.resetFormFields()
      this.clearDisable(this.treeData)
    },
    // 根据树节点key，禁用key对应的节点及其子节点,其他被禁用的节点恢复
    disableNodeWithChlidren(treeList, key ) {
      treeUtil.deepFirstTraverseTree({key: '0'}, treeList, (parentNode, node)=> {
        Object.assign(node, {disabled: false})
      })
      treeUtil.deepFirstTraverseTree({key: '0'}, treeList, (parentNode, node)=> {
        if(node.key === key || parentNode.disabled) {
          Object.assign(node, {disabled: true})
        }
      })
    },
    clearDisable(treeList) {
      treeUtil.deepFirstTraverseTree({key: '0'}, treeList, (parentNode, node)=> {
        Object.assign(node, {disabled: false})
      })
    },
    async getTreeDataAndSet() {
      const catalogNodes = await this.getDicCatalogs()
      if(catalogNodes) {
        catalogNodes.forEach(element => {
          element.value = element.key
          element.disabled = false
        });
        this.treeData = treeUtil.convertToTree({key: '0'}, catalogNodes)
      }
    },
    async getDicCatalogs() {
      const url = `/sys-mgt/sys-catalogs/nodes/type/7`
      const res = await this.$axios.$get(url)
      if(res.isSuccessed) {
        return res.data
      } else {
        this.$message.error(`查询失败,原因: ${res.message}`)
        return null
      }
    },
    handleBack() {
      this.addOrEditForm.resetFields()
      this.$emit('back')
    }
  }
}
</script>

<style  lang="less" scoped>
button {
  width: 80px;
  font-size: 12px;
  cursor: pointer;
}
.form-foot {
  position: relative;
  left: 30%;
}
.sysbtn-style {
  text-align: center;
  margin: 10px auto;
  width: 100px;
  font-size: 12px;
  cursor: pointer;
}

</style>
