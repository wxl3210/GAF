<template>
  <div>
    <a-form :form="form" layout="horizontal">
      <a-form-item
        v-show="menuType === '1'"
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="菜单名称"
      >
        <a-input-group compact>
          <a-select v-model="menuType" style="width: 100px">
            <a-select-option value="1">GAF平台</a-select-option>
            <a-select-option value="2">自定义</a-select-option>
          </a-select>
          <a-select
            v-decorator="[
              'name',
              {
                rules: [
                  {
                    required: true,
                    message: '请选择/检索菜单名称!',
                  },
                ],
              },
            ]"
            :filter-option="filterOption"
            placeholder="请选择/检索一个菜单"
            show-search
            style="width: 327.5px"
          >
            <a-select-option
              v-for="m of menusLang"
              :key="m.name"
              :value="m.name"
            >
              {{ m.value }}
            </a-select-option>
          </a-select>
        </a-input-group>
      </a-form-item>
      <a-form-item
        v-show="menuType === '2'"
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="菜单名称"
      >
        <a-input-group compact>
          <a-select v-model="menuType" style="width: 100px">
            <a-select-option value="1">GAF平台</a-select-option>
            <a-select-option value="2">自定义</a-select-option>
          </a-select>
          <a-input
            v-decorator="[
              'name',
              {
                rules: [
                  {
                    required: true,
                    message: '请输入菜单名称!',
                  },
                ],
              },
            ]"
            style="width: 327.5px"
          />
        </a-input-group>
      </a-form-item>
      <a-form-item
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="父级菜单"
      >
        <a-tree-select v-decorator="['pid']" :tree-data="menuList" />
      </a-form-item>
      <a-form-item :label-col="labelCol" :wrapper-col="valueCol" label="图标">
        <a-input
          v-decorator="[
            'icon',
            {
              rules: [
                {
                  required: true,
                  message: '请选择图标!',
                },
              ],
            },
          ]"
          disabled
          style="width: 200px"
        />
        <a-button icon="setting" @click="showIcon = true" />
      </a-form-item>
      <a-form-item :label-col="labelCol" :wrapper-col="valueCol" label="排序">
        <a-input-number v-decorator="['order']" :min="1" :max="1000" step="1" />
      </a-form-item>
      <a-form-item
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="是否可见"
      >
        <a-radio-group v-decorator="['visible']" name="radioGroup">
          <a-radio :value="true">是</a-radio>
          <a-radio :value="false">否</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item :label-col="labelCol" :wrapper-col="valueCol" label="路径">
        <a-input
          v-decorator="[
            'path',
            {
              rules: [
                {
                  required: true,
                  message: '请输入访问路径!',
                },
              ],
            },
          ]"
        />
      </a-form-item>
      <a-form-item
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="是否嵌入地址"
      >
        <a-radio-group
          v-decorator="['embed']"
          name="radioGroup"
          @change="(e) => handleRadioChange('embed', e)"
        >
          <a-radio :value="true">是</a-radio>
          <a-radio :value="false">否</a-radio>
        </a-radio-group>
      </a-form-item>
      <a-form-item
        v-if="embed"
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="嵌入地址"
      >
        <a-input
          v-decorator="[
            'embedUrl',
            {
              rules: [
                {
                  required: true,
                  message: '请输入嵌入地址!',
                },
              ],
            },
          ]"
        />
      </a-form-item>
      <a-form-item
        v-if="embed"
        :label-col="labelCol"
        :wrapper-col="valueCol"
        label="打开方式"
      >
        <a-radio-group
          v-decorator="['target']"
          name="radioGroup"
          @change="(e) => handleRadioChange('target', e)"
        >
          <a-radio value="0">直接门户打开</a-radio>
          <a-radio value="1">打开新页面</a-radio>
        </a-radio-group>
      </a-form-item>
    </a-form>
    <gaf-icon
      :visible="showIcon"
      @select="selectedIcon"
      @cancel="showIcon = false"
    />
  </div>
</template>

<script>
export default {
  name: 'AddRoleForm',
  props: {
    menuItem: {
      type: Object,
      required: false,
      default: () => null,
    },
    isNew: {
      type: Boolean,
      required: true,
    },
    menuList: {
      type: Array,
      required: true,
    },
  },
  data() {
    const routes = this.$store.getters.routes
    const menusLang = []
    const mkeys = Object.keys(routes)
    mkeys.forEach((mk) => {
      if (!mk.endsWith('_desc')) {
        menusLang.push({ name: mk, value: routes[mk] })
      }
    })
    return {
      menuType: '1',
      embed: false,
      target: '0',
      showIcon: false,
      menusLang,
      labelCol: { span: 5 },
      valueCol: { span: 18 },
    }
  },
  watch: {
    menuItem: {
      handler: 'handleMenuItemChange',
      immediate: true,
    },
  },
  beforeCreate() {
    this.form = this.$form.createForm(this)
  },
  methods: {
    filterOption(input, option) {
      return option.componentOptions.children[0].text
        .toLowerCase()
        .includes(input.toLowerCase())
    },
    selectedIcon(icon) {
      this.form.setFieldsValue({ icon })
      this.showIcon = false
    },
    handleRadioChange(name, e) {
      this[name] = e.target.value
      if (e.target.value) {
        if (name === 'embed') {
          this.$nextTick(function () {
            this.form.setFieldsValue({
              target: '0',
            })
          })
        }
      }
    },
    handleMenuItemChange(menuItem) {
      if (Object.keys(menuItem).length === 1) {
        this.addMenu(menuItem.pid)
      } else {
        this.editMenu(menuItem)
      }
    },
    addMenu(pid) {
      this.menuType = '1'
      this.$nextTick(function () {
        this.form.setFieldsValue({
          name: '',
          icon: '',
          order: 1,
          visible: true,
          pid,
        })
      })
    },
    editMenu(menuItem) {
      const {
        name,
        pid,
        path,
        icon,
        order,
        visible,
        embed,
        embedUrl,
        target,
      } = menuItem
      this.target = target
      this.embed = embed
      if (embed) {
        this.$nextTick(function () {
          this.form.setFieldsValue({
            name,
            pid,
            path,
            icon,
            order,
            visible,
            embed,
            embedUrl,
            target,
          })
        })
      } else {
        this.$nextTick(function () {
          this.form.setFieldsValue({
            name,
            pid,
            path,
            icon,
            order,
            visible,
            embed,
          })
        })
      }
    },
  },
}
</script>

<style scoped></style>
