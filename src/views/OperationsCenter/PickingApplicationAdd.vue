<template>
  <a-layout>
    <a-card>
      <a-form-model ref="ruleForm" :model="form" :rules="rules" :label-col="labelCol" :wrapper-col="wrapperCol">
        <a-form-model-item label="领料申请编码" required prop="PickingApplicationCode">
          <a-input v-model="form.PickingApplicationCode" placeholder="请输入领料申请单编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="elect">自动获取</a-button>
          </a-input>
        </a-form-model-item>

        <a-modal v-model="visible" title="选择编码" width="1000px" @ok="handleOk">
          <a-input-search placeholder="搜索" style="width: 400px;margin-bottom:20px" @search="onSearch" />
          <a-table :columns="columns" :data-source="data" :scroll="{ x: 1500 }" :pagination="false" bordered>
            <span slot="checked" style="margin: 0" slot-scope="text, record">
              <a-checkbox v-model="record.checked" @change="onChange(record)" />
            </span>
            <a slot="name" slot-scope="text">{{ text }}</a>
          </a-table>
        </a-modal>

        <a-form-model-item ref="name" label="部门编码" prop="DepartmentCode">
          <a-input v-model="form.DepartmentCode" placeholder="请输入部门编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
        </a-form-model-item>
        <a-form-model-item ref="name" label="业务员编码" prop="SalesmanCode">
          <a-input v-model="form.SalesmanCode" placeholder="请输入业务员编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
        </a-form-model-item>
        <a-form-model-item ref="name" label="预计出库日期" prop="ExpectedOutWarehouseDate">
          <a-date-picker
            v-model="form.ExpectedOutWarehouseDate"
            show-time
            type="date"
            placeholder="选择预计出库日期"
            style="width: 100%;"
          />
        </a-form-model-item>
        <a-form-model-item ref="Principal" label="商品清单" prop="Principal">
          <a-input v-model="form.Principal" placeholder="请选择存货编码" @blur="() => {}">
            <a-button slot="suffix" type="link" @click="showModal">选择</a-button>
          </a-input>
          <a-table
            :locale="{emptyText: '暂无数据'}"
            :columns="selectcolumns"
            :data-source="numberRow"
            :scroll="{ x: 1500 }"
            :pagination="false"
            bordered
          ></a-table>
        </a-form-model-item>
      </a-form-model>
    </a-card>
    <a-layout-footer :style="{ position: 'fixed',width: '100%',bottom: '0px',marginLeft: '-25px',zIndex:'999'}">
      <a-card>
        <a-row>
          <a-col :span='1' :offset='4'>
            <a-button type="primary" @click="resetForm">重置表单</a-button>
          </a-col>
          <a-col :span='1' :offset='1'>
            <a-button type="primary"  @click="onSubmit">保存</a-button>
          </a-col>
          <a-col :span='1' :offset='1'>
            <a-button type  @click="Back">返回</a-button>
          </a-col>
        </a-row>
      </a-card>
    </a-layout-footer>
  </a-layout>
</template>
<script>
import Vue from 'vue'
import { formModel, Button } from 'ant-design-vue'
Vue.use(formModel, Button)
import { postPickingApplicationAdd } from '@/api/manage'

const columns = [
  {
    title: '选择',
    dataIndex: 'checked',
    key: 'checked',
    width: 80,
    scopedSlots: { customRender: 'checked' }
  },
  {
    title: '领料申请单编码',
    dataIndex: 'PickingApplicationCode',
    width: 155,
    key: 'PickingApplicationCode',
    scopedSlots: { customRender: 'PickingApplicationCode' }
  },
  {
    title: '预计出库仓库编码',
    dataIndex: 'ExpectedOutWarehouseCode',
    width: 155,
    key: 'ExpectedOutWarehouseCode'
  },
  {
    title: '存货编码',
    width: 155,
    dataIndex: 'InventoryCode',
    key: 'InventoryCode'
  },
  {
    title: '存货名称',
    dataIndex: 'InventoryName',
    width: 155,
    key: 'InventoryName'
  },
  {
    title: '批次编码',
    dataIndex: 'BatchCode',
    width: 155,
    key: 'BatchCode'
  },
  {
    title: '数量',
    dataIndex: 'Quantity',
    width: 155,
    key: 'Quantity'
  },
  {
    title: '包装数量',
    dataIndex: 'PackingQuantity',
    width: 155,
    key: 'PackingQuantity'
  },
  {
    title: '包装单位',
    dataIndex: 'PackingUnit',
    width: 155,
    key: 'PackingUnit'
  },
  {
    title: '单价',
    dataIndex: 'UnitPrice',
    width: 155,
    key: 'UnitPrice'
  },
  {
    title: '金额',
    dataIndex: 'Amount',
    width: 155,
    key: 'Amount'
  }
]
const selectcolumns = [
  {
    title: '领料申请单编码',
    dataIndex: 'PickingApplicationCode',
    width: 155,
    key: 'PickingApplicationCode',
    scopedSlots: { customRender: 'PickingApplicationCode' }
  },
  {
    title: '预计出库仓库编码',
    dataIndex: 'ExpectedOutWarehouseCode',
    width: 155,
    key: 'ExpectedOutWarehouseCode'
  },
  {
    title: '存货编码',
    dataIndex: 'InventoryCode',
    width: 155,
    key: 'InventoryCode'
  },
  {
    title: '存货名称',
    dataIndex: 'InventoryName',
    width: 155,
    key: 'InventoryName'
  },
  {
    title: '批次编码',
    dataIndex: 'BatchCode',
    width: 155,
    key: 'BatchCode'
  },
  {
    title: '数量',
    dataIndex: 'Quantity',
    width: 155,
    key: 'Quantity'
  },
  {
    title: '包装数量',
    dataIndex: 'PackingQuantity',
    width: 155,
    key: 'PackingQuantity'
  },
  {
    title: '包装单位',
    dataIndex: 'PackingUnit',
    width: 155,
    key: 'PackingUnit'
  },
  {
    title: '单价',
    dataIndex: 'UnitPrice',
    width: 155,
    key: 'UnitPrice'
  },
  {
    title: '金额',
    dataIndex: 'Amount',
    width: 155,
    key: 'Amount'
  }
]

const data = [
  {
    key: '1',
    PickingApplicationCode: 'a121345',
    DepartmentCode: 'a121345',
    SalesmanCode: 'a121345',
    ExpectedOutWarehouseCode: 'a121345',
    InventoryCode: 'a121345',
    LocationCode: 'a121345',
    BatchCode: 'a121345'
  },
  {
    key: '2',
    PickingApplicationCode: 'a121345',
    DepartmentCode: 'a121345',
    SalesmanCode: 'a121345',
    ExpectedOutWarehouseCode: 'a121345',
    InventoryCode: 'a121345',
    LocationCode: 'a121345',
    BatchCode: 'a121345'
  },
  {
    key: '3',
    PickingApplicationCode: 'a121345',
    DepartmentCode: 'a121345',
    SalesmanCode: 'a121345',
    ExpectedOutWarehouseCode: 'a121345',
    InventoryCode: 'a121345',
    LocationCode: 'a121345',
    BatchCode: 'a121345'
  }
]
const numberRow = []
export default {
  data() {
    return {
      numberRow,
      selectedRow: [],
      selectcolumns,
      visible: false,
      selectedRowKeys: [],
      data,
      columns,
      headers: {
        authorization: 'authorization-text'
      },
      size: 'small',
      labelCol: { span: 2 },
      wrapperCol: { span: 22 },
      other: '',
      form: {
        PickingApplicationCode: '', //领料申请单编码
        DepartmentCode: '', //部门编码
        SalesmanCode: '',
        ExpectedOutWarehouseDate: '', //日期
        ExpectedOutWarehouseCode: '', //预计出库仓库编码
        InventoryCode: '', //存货编码
        InventoryName: '', //存货名称
        BatchCode: '', //批次编码
        Quantity: '', //数量
        PackingQuantity: '', //包装数量
        PackingUnit: '', //包装单位
        UnitPrice: '', //单价
        Amount: '' //金额
      },
      rules: {
        PickingApplicationCode: [{ required: true, message: '请输入产品编码', trigger: 'blur' }],
        region: [{ required: true, message: '', trigger: 'change' }],
        date1: [{ required: true, message: '', trigger: 'change' }],
        type: [
          {
            type: 'array',
            required: true,
            message: 'Please select at least one activity type',
            trigger: 'change'
          }
        ],
        resource: [{ required: true, message: 'Please select activity resource', trigger: 'change' }],
        desc: [{ required: true, message: '请输入产品说明', trigger: 'blur' }]
      }
    }
  },
  computed: {
    rowSelection() {
      const { selectedRowKeys } = this
      return {
        selectedRowKeys,
        onChange: this.onSelectChange,
        hideDefaultSelections: true,
        onSelection: this.onSelection
      }
    }
  },
  methods: {
    onSearch(value) {
      console.log(value)
    },
    handleChange(info) {
      if (info.file.status !== 'uploading') {
        console.log(info.file, info.fileList)
      }
      if (info.file.status === 'done') {
        this.$message.success(`${info.file.name} file uploaded successfully`)
      } else if (info.file.status === 'error') {
        this.$message.error(`${info.file.name} file upload failed.`)
      }
    },
    handleSearchChange(value) {
      console.log(`selected ${value}`)
    },
    onSubmit() {
      this.$refs.ruleForm.validate(valid => {
        console.log('name--->', this.form)
        if (valid) {
          postPickingApplicationAdd(this.form).then(res => {
            console.log('res----->', res)
          })
          alert('保存成功，点击确认回到档案界面!')
          this.$router.push({ name: 'PickingApplicationList' })
        } else {
          console.log('error submit!!')
          alert('亲，您的填写内容不符合要求，请重新填写！！！')
          return false
        }
      })
    },
    Back() {
      this.$router.push({ name: 'PickingApplicationList' })
    },
    resetForm() {
      this.$refs.ruleForm.resetFields()
    },
    handleBlur() {
      console.log('blur')
    },
    handleFocus() {
      console.log('focus')
    },
    filterOption(input, option) {
      return option.componentOptions.children[0].text.toLowerCase().indexOf(input.toLowerCase()) >= 0
    },
    onSelectChange(selectedRowKeys) {
      this.selectedRowKeys = selectedRowKeys
    },
    elect() {
      this.form.PickingApplicationCode = 'PT2020062200001'
    },
    showModal() {
      this.visible = true
    },

    handleOk(e) {
      console.log(e)
      this.visible = false
      this.numberRow = this.selectedRow
      console.log(this.numberRow)
    },
    onChange(record) {
      console.log('check', record)
      if (record.checked) {
        this.selectedRow.push(record)
        console.log(this.selectedRow)
      }
    }
  }
}
</script>
