<template>
  <div>
    <template>
      <div style="margin: 7px 0">
        <a-breadcrumb separator=">">
          <a-breadcrumb-item class="tree-catalog" style="line-height: 15px"
            ><span class="vertical-line">| </span>{{ title }}</a-breadcrumb-item
          >
        </a-breadcrumb>
      </div>
    </template>
    <div>
      <a-form
        style="padding: 15px 20%"
        :form="addOrEditForm"
        :label-col="{ span: 5 }"
        :wrapper-col="{ span: 13 }"
        layout="horizontal"
      >
        <a-form-item v-show="false" label="目录分类">
          <a-input
            v-decorator="[
              'type',
              {
                rules: [
                  {
                    required: true,
                  },
                ],
                initialValue: '0',
              },
            ]"
            allow-clear
            placeholder="请选择类别"
          >
            <a-select-option value="0">无</a-select-option>
          </a-input>
        </a-form-item>
        <a-form-item v-show="!isFirstLevel" label="上级目录">
          <a-cascader
            v-decorator="[
              'parentPath',
              {
                rules: [
                  {
                    required: true,
                    message: '请选择上级目录',
                  },
                ],
                initialValue: ['0'],
              },
            ]"
            :options="options"
            :field-names="{
              label: 'title',
              value: 'key',
              children: 'children',
            }"
            :disabled="casecaderDisabled"
            placeholder="请选择上级目录"
            change-on-select
            allow-clear
            @change="onChange"
          />
        </a-form-item>
        <a-form-item v-show="false" label="上级目录id">
          <a-input
            v-decorator="[
              'parentId',
              {
                rules: [
                  {
                    required: true,
                  },
                ],
                initialValue: '0',
              },
            ]"
          />
        </a-form-item>
        <a-form-item label="目录名称">
          <a-input
            v-decorator="[
              'name',
              {
                rules: [
                  {
                    required: true,
                    message: '请输入目录名称',
                  },
                ],
              },
            ]"
            placeholder="请输入目录名称"
            allow-clear
          />
        </a-form-item>
        <a-form-item v-show="isFirstLevel" label="业务类别">
          <a-select
            v-decorator="['bizTypeCode', { initialValue: '1' }]"
            placeholder="请选择业务类别"
            :options="bizTypes"
            :disabled="operation === 'edit'"
          ></a-select>
        </a-form-item>
        <a-form-item label="排序序号">
          <a-input-number
            v-decorator="['sortSn']"
            :disabled="operation === 'detail'"
            :precision="0"
            :min="1"
          />
        </a-form-item>
        <a-form-item v-show="false" label="图标地址">
          <a-input
            v-decorator="['iconUrl']"
            :disabled="operation === 'detail'"
            placeholder="请输入图标地址"
            allow-clear
          />
        </a-form-item>
        <a-form-item label="描述">
          <a-textarea
            v-decorator="['description']"
            :disabled="operation === 'detail'"
            placeholder="请输入描述"
            auto-size
          />
        </a-form-item>
        <div v-if="operation === 'detail'">
          <a-form-item label="创建时间">
            <a-date-picker
              v-decorator="['createdTime']"
              show-time
              placeholder="请选择创建时间"
              disabled
            />
          </a-form-item>
          <a-form-item label="创建人">
            <a-input
              v-decorator="['createdBy']"
              placeholder="请输入创建人"
              allow-clear
              disabled
            />
          </a-form-item>
          <a-form-item label="修改时间">
            <a-date-picker
              v-decorator="['updatedTime']"
              show-time
              placeholder="请选择修改时间"
              disabled
            />
          </a-form-item>
          <a-form-item label="修改人">
            <a-input
              v-decorator="['updatedBy']"
              placeholder="请输入修改人"
              allow-clear
              disabled
            />
          </a-form-item>
        </div>
        <div style="text-align: center; margin-top: 15px">
          <!-- <button class="cancel-modal" v-if="!isFirstLevel && this.operation === 'edit'" @click="deleteDataOrCancle">{{
            deleteOrCacel
          }}</button> -->
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          <button class="submit-gray" style="color: white" @click="submitForm">
            {{ submitButtonText }}
          </button>
          &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
          <button class="cancel-modal" v-if="!isFirstLevel && this.operation === 'add'" @click="deleteDataOrCancle">{{
            deleteOrCacel
          }}</button>
        </div>
      </a-form>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    // 当parentId为'0'或为空时表示一级分类目录
    editData: {
      type: Object,
      default: null,
    },
    // add表示新增 edit表示表示编辑 detail表示详情
    operation: {
      type: String,
      default: 'edit',
    },
    options: {
      type: Array,
      default: () => [
        {
          key: '0',
          title: '无',
        },
      ],
    },
    // 控制表单不同名字的显示
    catalogType: {
      type: String,
      default: '0',
    },
    bizTypes: {
      type: Array,
      default: () => [],
    },
  },
  data() {
    return {
      dataId: '',
      casecaderDisabled: true,
    }
  },
  computed: {
    deleteOrCacel() {
      if (this.operation === 'edit') {
        return '删除'
      } else if (this.operation === 'add') {
        return '取消'
      } else {
        return ''
      }
    },
    submitButtonText() {
      if (this.operation === 'edit') {
        return '保存'
      } else if (this.operation === 'add') {
        return '添加'
      } else {
        return ''
      }
    },
    title() {
      if (this.operation === 'edit') {
        if (!this.isFirstLevel) {
          if (this.catalogType === '1') {
            return '编辑模块分组'
          } else if (this.catalogType === '2') {
            return '编辑API分组'
          } else if (this.catalogType === '3') {
            return '编辑角色分组'
          } else if (this.catalogType === '4') {
            return '编辑菜单分组'
          } else if (this.catalogType === '6') {
            return '编辑目录'
          } else {
            return '编辑子目录'
          }
        } else {
          return '编辑根目录'
        }
      } else if (this.operation === 'add') {
        if (!this.isFirstLevel) {
          if (this.catalogType === '1') {
            return '添加模块分组'
          } else if (this.catalogType === '2') {
            return '添加API分组'
          } else if (this.catalogType === '3') {
            return '添加角色分组'
          } else if (this.catalogType === '4') {
            return '添加菜单分组'
          } else if (this.catalogType === '6') {
            return '添加目录'
          } else {
            return '添加子目录'
          }
        } else {
          return '添加根目录'
        }
      } else {
        return ''
      }
    },
    isFirstLevel() {
      return this.editData.parentId === '0'
    },
  },
  watch: {
    editData(newEditData) {
      this.addOrEditForm.resetFields()
      this.dataId = newEditData.catalogId
      delete newEditData.catalogId
      delete newEditData.code
      delete newEditData.status
      delete newEditData.createdTime
      delete newEditData.createdBy
      delete newEditData.updatedTime
      delete newEditData.updatedBy
      delete newEditData.tenantId
      delete newEditData.sysComponentId
      if (newEditData.parentPath != null) {
        newEditData.parentId =
          newEditData.parentPath[newEditData.parentPath.length - 1]
      }
      this.addOrEditForm.setFieldsValue({ ...newEditData })
    },
  },
  beforeMount() {
    this.addOrEditForm = this.$form.createForm(this, { name: 'addOrEditForm' })
  },
  mounted() {
    this.addOrEditForm.resetFields()
    this.dataId = this.editData.catalogId
    delete this.editData.catalogId
    delete this.editData.code
    delete this.editData.status
    delete this.editData.createdTime
    delete this.editData.createdBy
    delete this.editData.updatedTime
    delete this.editData.updatedBy
    delete this.editData.tenantId
    delete this.editData.sysComponentId
    if (this.editData.parentPath != null) {
      this.editData.parentId = this.editData.parentPath[
        this.editData.parentPath.length - 1
      ]
    }
    this.addOrEditForm.setFieldsValue({ ...this.editData })
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
          const { name, sortSn, iconUrl, description, parentId } = data
          const rst = await this.$axios.put(url, {
            name,
            sortSn,
            iconUrl,
            description,
            parentId,
          })
          if (rst.data.isSuccessed) {
            this.$message.success('更新成功')
            this.$emit('update-success', rst.data.data)
          } else {
            this.$message.error(`更新失败,原因:${rst.data.message}`)
          }
        } else {
          const rst = await this.$axios.post(url, data)
          if (rst.data.isSuccessed) {
            this.$message.success('添加成功')
            this.$emit('add-success', rst.data.data)
          } else {
            this.$message.error(`添加失败,原因:${rst.data.message}`)
          }
        }
      })
    },
    async deleteDataOrCancle() {
      if (this.operation === 'edit') {
        if (this.dataId) {
          const url = `/sys-mgt/sys-catalogs/` + this.dataId
          const rst = await this.$axios.delete(url)
          if (rst.data.isSuccessed) {
            this.$message.success('删除成功')
            this.$emit('delete-success', rst.data.data)
          } else {
            this.$message.error('删除失败，原因: ' + rst.data.message)
          }
        }
      } else if (this.operation === 'add') {
        this.$emit(
          'cancleWhenAdd',
          this.addOrEditForm.getFieldValue('parentId')
        )
      }
    },
    onChange(path) {
      const parentId = path[path.length - 1]
      this.addOrEditForm.setFieldsValue({ parentId })
    },
    resetFormFields() {
      this.addOrEditForm.resetFields()
    },
    setFormFields(fieldsValue) {
      if (fieldsValue) {
        this.addOrEditForm.setFieldsValue(fieldsValue)
      }
    },
  },
}
</script>

<style scoped>
button {
  width: 100px;
  font-size: 12px;
  cursor: pointer;
}
</style>
