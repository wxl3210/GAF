<template>
  <div class="app-container">
    <div class="page-single">
    <gaf-table-layout>
      <template #actions>
        <button @click="handleAdd" class="btn-fun blue">
          <span><a-icon type="plus" />
          新建目录</span>
        </button>
        <!-- <button @click="batchDel" class="btn-fun red">
          <span><a-icon type="delete" />
          批量删除</span>
        </button> -->
      </template>
      <!-- <template #filter>
        <div style="margin-top: 5px">
          <a-input-search
            @search="onSearch"
            style="width: 320px"
            :placeholder="searchedColumn === 'dict_name'? '请输入名称模糊查询': '请输入字典值查询'"
            size="large"
          >
            <button slot="enterButton" class="btn-search">
              搜索
            </button>
          </a-input-search>
        </div>
      </template> -->
      <template #default>
        <gaf-table-with-page
          :showXH="false"
          :scroll="{x: 1500 , y: 570}"
          :data-source="dataList"
          :loading="loading"
          @change="tableChange"
          @expand="onExpand"
          :expandedRowKeys="expandedRowKeys"
          :row-key="r => r.name"
          :columns="columns"
          style="width: 100%;"
          bordered
          size="small"
        >
          <template slot="name" slot-scope="text">
            {{ getName(text) }}
          </template>
          <template v-if="timeFormat" slot="timeRender" slot-scope="text">
            {{ timeFormat(text) }}
          </template>
          <template
            slot="operation"
            slot-scope="text, record"
          >
            <a @click.stop="() => handleAddChild(record)" class="btn-code" href="javascript:;" v-if="record.type === 'commonPrefix'">
              <a-icon type="plus" /> 新建子目录
            </a>
            
            <a-divider type="vertical" v-if="record.type === 'commonPrefix'" />
            <a @click.stop="() => handleDownload(record)" v-if="record.type === 'object'" class="btn-edit" href="javascript:;">
              <a-icon type="edit" /> 下载
            </a>

            <a-divider v-if="record.type === 'object'" type="vertical" />
            <a-popconfirm
              @confirm="() => handleDelete(record)"
              class="btn-del"
              title="删除后无法恢复，确认是否继续?"
              ok-text="确认"
              cancel-text="取消"
            >
              <a href="javascript:;"><a-icon type="delete" /> 删除</a>
            </a-popconfirm>
            <a-divider type="vertical" v-if="record.type === 'commonPrefix'" />
            <gaf-upload text="上传" :showUploadList="false" :dir="record.name" minioServiceUrl="/storage/file-storage/" class="btn-upload" :style="uploadStyle" @uploadComplate="uploadComplate" v-if="record.type === 'commonPrefix'"></gaf-upload>
          </template>

          <template v-if="timeFormat" slot="timeRender" slot-scope="text">
            {{ timeFormat(text) }}
          </template>
          <template slot="visibility" slot-scope="text">
            {{text ? '可见': '不可见'}}
          </template>
        </gaf-table-with-page>
      </template>
    </gaf-table-layout>
	<a-modal
    v-model="open"
    :width="800"
    :footer="null"
    :centered="true"
    @cancel="handleBack"
    destroy-on-close
  >
    <add-edit-form
      :title="title"
      :editData="editData"
      @submit="handleSubmit"
      @back="handleBack"
      :operation="operation"
      :name="name"
    >
    </add-edit-form>
  </a-modal>
  </div>
  </div>
</template>

<script>
import AddEditForm from '../../views/AttachmentManagement/AddOrEditForm'
import {gafDownloadUtil} from 'gaf-ui'

export default {
  components: {
    AddEditForm
  },
  props: {
    dicTypeId: {
      type: String,
      default: null
    },
    dictCode: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      expandedRowKeys: [],
      name:'',
      expanded: true,
      uploadStyle: 'height: auto,background-color: rgb(221, 171, 92);border: 1px solid transparent;padding: 2px 5px;border-radius: 5px;color: #fff;',
      // 标题
      title: '',
      // 编辑记录
      editData: [],
      selectedRowKeys: [],
      // ${functionName}表格数据
      dataList: [],
      // 是否显示添加修改弹出层
      open: false,
      // 分页参数
      pagination: {
        pageSize: 10,
        current: 1,
        total: 0
      },
      // 列表是否加载中
      loading: false,
      searchText: '',
      searchedColumn: 'dict_name',
      sorter: {
        order: 'ASC',
        field: 'seq'
      },
      // 详情：1，新增：2，编辑：3
      operation: 0,
      columns: [
        {
          title: '名称',
          dataIndex: 'name',
          key: 'name',
          width: 600 ,
          align: 'left',
          scopedSlots: { customRender: 'name' },
        },
        {
          title: '大小(KB)',
          dataIndex: 'size',
          key: 'size',
          width: 125,
          align: 'center',
        },
        {
          title: '时态',
          dataIndex: 'lastModified',
          key: 'lastModified',
          width: '250px',
          scopedSlots: { customRender: 'timeRender' },
        },
        {
          title: '操作',
          key: 'operation',
          scopedSlots: { customRender: 'operation' },
          fixed: 'right'
        }
      ]
    }
  },
  watch: {
    dicTypeId: function () {
      this.pagination.current = 1
      this.getList()
    }
  },
  computed: {
    timeFormat: function() {
      if (
        this.columns.filter(
          item =>
            item.scopedSlots && item.scopedSlots.customRender === 'timeRender'
        ).length > 0
      ) {
        return function(str) {
          if (!str || str === '') {
            return ''
          }
          const dt = new Date(str)
          const year = dt.getFullYear()
          let month = dt.getMonth() + 1
          let date = dt.getDate()
          let hour = dt.getHours()
          let minute = dt.getMinutes()
          let second = dt.getSeconds()

          month = month < 10 ? '0' + month : month
          date = date < 10 ? '0' + date : date
          hour = hour < 10 ? '0' + hour : hour
          minute = minute < 10 ? '0' + minute : minute
          second = second < 10 ? '0' + second : second

          return `${year}/${month}/${date} ${hour}:${minute}:${second}`
        }
      }
      return null
    }
  },
  created() {
    this.getList()
  },
  methods: {
    handleSearchFieldChange(value) {
      this.searchedColumn = value
    },
    // 搜索查询
    async onSearch(val) {
      this.searchText = val
      this.pagination.current = 1
      await this.getList()
    },
    // 页码，排序项发生改变时，重新获取列表数据
    tableChange(pageInfo, filters, sorter) {
      if (pageInfo) {
        this.pagination.current = pageInfo.current
        this.pagination.pageSize = pageInfo.pageSize
      }
      if (sorter.order) {
        this.sorter.order = sorter.order === 'descend' ? 'DESC' : 'ASC'
        this.sorter.field = sorter.columnKey
      } else {
        this.sorter.order = null
        this.sorter.field = null
      }
      this.getList()
    },
    async onExpand(expanded, record) {
      
      if(expanded && !record.expanded) {
        
        this.loading = true
        let url = `/storage/file-storage/list-objects?prefix=${record.name}`
        const res = await this.$axios.$get(url)
        if (res.isSuccessed) {
          this.expanded = false
          this.loading = false
          res.data.forEach(element => {
            if(element.type === 'commonPrefix') {
              element.children = []
            }
          })
          record.expanded = true
          record.children.push(...res.data)
          // this.dataList.forEach(item => {
          //   if (item.name === record.name){
          //     item.children.push(...res.data)
          //   }
          // })
        } else {
          this.$message.error(`查询失败,原因:${res.message}`)
        }
      }
      if (expanded) {
        this.expandedRowKeys.push(record.name)
      }else {
        this.expandedRowKeys.splice(this.expandedRowKeys.indexOf(record.name),1)
      }
    },
    // onExpandedRowsChange(expandedRows) {
    //   if(!expandedRows || expandedRows.length == 0) {
    //     this.expandedRowKeys = []
    //   } else {
    //     this.expandedRowKeys = expandedRows.map(el=> el.dictId);
    //   }
    // },
    // 添加数据
    handleAdd() {
      this.operation = 2
      this.open = true
      this.editData = this.dataList
      this.title = '新建目录'
    },
    // 添加修改提交后
    handleSubmit() {
      this.dataList = [...this.dataList]
      this.open = false
    },
    // 添加修改返回后
    handleBack() {
      this.editData = []
      this.open = false
    },
    // 下载
    handleDownload(row) {
      gafDownloadUtil(this.$axios,'/storage/file-storage/',row.name)
    },
    handleAddChild(row) {
      this.operation = 2
      this.open = true
      this.editData = row.children
      this.name = row.name
      this.title = '新建子目录'
    },
    handleDetail(row) {
      this.operation = 1
      this.open = true
      this.title = '详情展示'
      this.editData =  {dataDictId:row.key, pid: row.parentId, dictCode: this.dictCode, dictName: row.label, 
        dictValue: row.value, seq: row.seq, visibility: row.visibility, dictDesc: row.dictDesc,
        createdTime: row.createdTime, createdBy: row.createdBy, updatedTime: row.updatedTime,
        updatedBy: row.updatedBy,extProperties: row.extProperties
      }
    },
    // 删除数据
    async handleDelete(row) {
      const url = `/storage/file-storage/` + row.name
      const rst = await this.$axios.delete(url)
      if (rst.data.isSuccessed) {
        this.$message.success('删除成功')
      } else {
        this.$message.error(`删除失败,原因:${rst.data.message}`)
      }
      this.$nextTick(() => {
        
        this.getList()
        this.expandedRowKeys = []
      })
    },
    // 批量删除
    async batchDel() {
      const url = `/storage/file-storage/`
      const selectedRowKeys = this.selectedRowKeys
      if (selectedRowKeys.length !== 0) {
        const rst = await this.$axios.delete(url, { data: selectedRowKeys })
        if (rst.data.isSuccessed) {
          this.$message.success('删除成功')
        } else {
          this.$message.error(`删除失败,原因:${rst.data.message}`)
        }
        this.$nextTick(() => {
          this.pagination.current = 1
          this.getList()
        })
      } else {
        this.$message.warn('请选择您要删除的内容')
      }
    },
    // 复选框
    rowChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    rowSelect(record, selected, selectedRows) {
      // eslint-disable-next-line no-console
      console.log(record, selected, selectedRows)
    },
    rowSelectAll(selected, selectedRows, changeRows) {
      // eslint-disable-next-line no-console
      console.log(selected, selectedRows, changeRows)
    },
    async getList() {
      this.loading = true
      let url = `/storage/file-storage/list-objects`
      if (this.searchText.trim() && this.searchedColumn) {
        url =
          url +
          '&searchFieldName=' +
          this.searchedColumn +
          '&searchFieldValue=' +
          this.searchText.trim()
      }
      // if (this.sorter.order && this.sorter.field) {
      //   url =
      //     url +
      //     '&orderFieldName=' +
      //     this.sorter.field +
      //     '&orderMethod=' +
      //     this.sorter.order
      // }
      const res = await this.$axios.$get(url)
      this.loading = false
      if (res.isSuccessed) {
        this.pagination.current = res.data.pageIndex
        this.pagination.pageSize = res.data.pageSize
        this.pagination.total = res.data.total
        // children空数组置为null
        // if(res.data.content && res.data.content.length > 0) {
        //   treeUtil.deepFirstTraverseTree(null, res.data.content, (parentNode, node) => {
        //     if(node.children && node.children.length == 0) {
        //       node.children = null
        //     }
        //   })
        // }
        this.dataList = res.data
        this.dataList.forEach(element => {
          if(element.type === 'commonPrefix') {
            element.children = []
          }
        });
      } else {
        this.$message.error(`查询失败,原因:${res.message}`)
      }
    },
    uploadComplate() {
      this.getList()
      this.expandedRowKeys = []
    },
    getName(data) {
      let name = data.split("/")
      if (name[name.length-1] === ''){
        return name[name.length-2] + '/'
      } else {
        return name[name.length-1]
      }
    }
  }
}
</script>

<style scoped lang="less">
.app-container {
  height: 100%;
}
.btn-upload {
  display: inline-block;
  button {
    height: auto;
    background-color: rgb(221, 171, 92);
    border: 1px solid transparent;
    padding: 2px 5px;
    border-radius: 5px;
    color: #fff;
  }
}
</style>
